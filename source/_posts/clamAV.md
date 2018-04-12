---
title: 记 ClamAV 在MAC OS上 安装过程
copyright: true
date: 2018-04-12 10:27:03
tags: ClamAV
categories: 开发
---

最近因为项目的需要用到了[ClamAV](https://www.clamav.net/), 在Mac OS上安装时遇到了一些问题，特此记录一下安装 [ClamAV](https://www.clamav.net/) 的坑

> ClamAV 杀毒是Linux平台最受欢迎的杀毒软件，ClamAV属于免费开源产品，支持多种平台，如：Linux/Unix、MAC OS X、Windows、OpenVMS。ClamAV是基于病毒扫描的命令行工具，但同时也有支持图形界面的ClamTK工具。ClamAV主要用于邮件服务器扫描邮件。它有多种接口从邮件服务器扫描邮件，支持文件格式有如：ZIP、RAR、TAR、GZIP、BZIP2、HTML、DOC、PDF,、SIS CHM、RTF等等。ClamAV有自动的数据库更新器，还可以从共享库中运行。命令行的界面让ClamAV运行流畅


# 安装步骤

1. Open terminal, 使用 [Homebrew](https://brew.sh/) 来安装 [ClamAV](https://www.clamav.net/)
  ```bash
  $ brew install clamav
  ```
2. 配置 `freshclam.conf`， 因为安装完 clamav之后， 在目录 `/usr/local/etc/clamav`下存在一个 `freshclam.conf.sample`的文件，建议不要去重命名或者Copy一份，最快的方式是我们自己创建一个新的`freshclam.conf`文件。
  ```bash
  $ cd /usr/local/etc/clamav
  $ touch freshclam.conf
  $ vi freshclam.conf
  ```
  然后在`freshclam.conf`文件中写如以下内容 (就是去下载病毒库的地址)
  ```bash
  DatabaseMirror database.clamav.net
  ```
3. 配置 `clamd.conf`, 同上一步一样，我们自己创建一个`clamd.conf` 文件
  ```bash
  $ cd /usr/local/etc/clamav
  $ touch clamd.conf
  $ vi clamd.conf
  ```
  然后在`clamd.conf` 文件中写入以下内容
  ```bash
  TCPSocket 3310
  LocalSocket /usr/local/var/run/clamav/clamd.sock
  ```
  * **TCPSocket** 是端口号, 一般配置 3310, 当然根据你自己机器使用的情况可自行定义
  * **LocalSocket** 本机Socket路径, 这样我们配置是在 `/usr/local/var/run/clamav/`目录下，但到目前为止，机器上不存在这个目录，因为我们需要手支创建
    ```bash
    $ mkdir -p /usr/local/var/run/clamav/
    ```

好了，现在所有的配置都已经完成，可以试着更新病毒库，并启动`clamd`服务

# 如何使用
1. 在运行之前我们必须要下载clamAV 的病毒库, run `freshclam`命令即可
  ```bash
  $ freshclam -v
  ```
  显示如下信息:
  ```bash
  Current working dir is /usr/local/Cellar/clamav/0.98.1/share/clamav
  Max retries == 3
  ClamAV update process started at Tue Feb  4 11:31:22 2014
  Using IPv6 aware code
  Querying current.cvd.clamav.net
  TTL: 1694
  Software version from DNS: 0.98.1
  Retrieving http://database.clamav.net/main.cvd
  Trying to download http://database.clamav.net/main.cvd (IP: 81.91.100.173)
  Downloading main.cvd [100%]
  Loading signatures from main.cvd
  Properly loaded 2424225 signatures from new main.cvd
  main.cvd updated (version: 55, sigs: 2424225, f-level: 60, builder: neo)
  Querying main.55.76.1.0.515B64AD.ping.clamav.net
  ...
  ```
2. 当所有的下载完成之后，现在你肯定迫不及待的想启动 `clamd`服务了，在terminal中输入以下命令
  ```bash
  $ clamd
  ```
  如果你看到的是如下错误信息，那么很"荣幸"的告诉你，你已经在"坑"里了
  ```bash
  clamd is not found
  ```
  **根本原因**

  `clamd` 是在 `/usr/local/sbin/`目录下：
  ```bash
  $ cd /usr/local/sbin/
  $ ls -al
  ```
  可以看到 `clamd`的确在`/usr/local/sbin/`目录下
  ```bash
  clamd -> ../Cellar/clamav/0.99.4/sbin/clamd
  ```

  **解决办法**

  将`clamd`copy到 `/usr/local/bin`目录下，如何Copy, 请参照以下命令
  ```bash
  $ cd /usr/local/bin
  # 创建 clamd 的 soft link， 用 ln命令来创建 soft link
  # ln sourcePath targetPath
  $ sudo ln -s ../Cellar/clamav/0.99.4/sbin/clamd clamd
  ```
  以在命令将在`/usr/local/bin`目录下创建 `clamd`命令的 link, 运行了以上命令之后再次在 terminal中运行 `clamd`
  ```bash
  $ clamd
  ```
  检查一下`clamd`是否已经启动了
  ```bash
  $ ps -ef | grep clamd
  ```
  查看 `clamd`是在哪个端口号命令如下:
  ```bash
  $ lsof -Pnl +M | grep clamd
  ```
  可得到如下结果:
  ```bash
  clamd     40280      501  cwd       DIR                1,4       1258         2 /
  clamd     40280      501  txt       REG                1,4     165196 145399832 /usr/local/bin/clamd
  clamd     40280      501  txt       REG                1,4    1689968 145399833 /usr/local/Cellar/clamav/0.99.4/lib/libclamav.7.dylib
  clamd     40280      501  txt       REG                1,4     385168 143495550 /usr/local/Cellar/openssl/1.0.2n/lib/libssl.1.0.0.dylib
  clamd     40280      501  txt       REG                1,4    1991920 143495547 /usr/local/Cellar/openssl/1.0.2n/lib/libcrypto.1.0.0.dylib
  clamd     40280      501  txt       REG                1,4      13804 145399837 /usr/local/Cellar/clamav/0.99.4/lib/libclamunrar_iface.7.so
  clamd     40280      501  txt       REG                1,4     423664 145399741 /usr/local/Cellar/pcre/8.41/lib/libpcre.1.dylib
  clamd     40280      501  txt       REG                1,4      45832 145399835 /usr/local/Cellar/clamav/0.99.4/lib/libclamunrar.7.dylib
  clamd     40280      501  txt       REG                1,4     698896 132159863 /usr/lib/dyld
  clamd     40280      501  txt       REG                1,4  662388736 146709470 /private/var/db/dyld/dyld_shared_cache_x86_64h
  clamd     40280      501    0r      CHR                3,2        0t0       304 /dev/null
  clamd     40280      501    1w      CHR                3,2     0t1012       304 /dev/null
  clamd     40280      501    2w      CHR                3,2        0t0       304 /dev/null
  clamd     40280      501    3u    systm                           0t0           
  clamd     40280      501    4u     IPv6 0x48a0431076b7a18b        0t0       TCP *:3310 (LISTEN)
  clamd     40280      501    5u     IPv4 0x48a0431084192923        0t0       TCP *:3310 (LISTEN)
  clamd     40280      501    6                                                   (revoked)
  clamd     40280      501    7u     unix 0x48a043107b7fc18b        0t0           /usr/local/var/run/clamav/clamd.sock
  clamd     40280      501    8      PIPE 0x48a04310982babeb      16384           ->0x48a04310982bbaeb
  clamd     40280      501    9      PIPE 0x48a04310982bbaeb      16384           ->0x48a04310982babeb
  clamd     40280      501   10      PIPE 0x48a04310982bab2b      16384           ->0x48a04310982bc26b
  clamd     40280      501   11      PIPE 0x48a04310982bc26b      16384           ->0x48a04310982bab2b
  ```
  **TCP *:3310 (LISTEN)** 3310便是 `clamd`的端口号

# 总结
好了，经过上面的配置步骤，我们已经可以成功的 start `clamd` service, 并且知道能过 `ps` 命令来检查 `clamd`是否启动。

同时我们也可以通过 `lsof` + `grep` 命令来查看 `clamd`运行在哪个端口下。

Enjoin, have ClamAV funny!!

# ClamAV 与Java集成
请参考
* [clamav-java GitHub](https://github.com/solita/clamav-java)
* [clamav-rest GitHub](https://github.com/solita/clamav-rest)
-------
<span style="font-size=10px">
参考： https://gist.github.com/zhurui1008/4fdc875e557014c3a34e
</span>
