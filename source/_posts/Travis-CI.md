---
title: 使用Travis-CI来自动部署hexo博客
date: 2018-03-16 14:26:54
tags:
  - 自动部署hexo

categories:
  - Hexo搭建博客
---

> 本文介绍了如何使用[Travis CI](https://travis-ci.org)自动部署hexo，解决了家里和公司两地写博客的麻烦。本博客实现了博客仓库中的dev分支改动时，自动部署到master分支中。
> [Travis CI](https://travis-ci.org) 是在软件开发领域中的一个在线的，分布式的持续集成服务，用来构建及测试在GitHub托管的代码
> 最重要的一点 [Travis CI](https://travis-ci.org) 是 **免费**, **免费**, **免费** 的

# 前提条件
- 你已经有了[GitHub](https://github.com/) 账号
- 你已经有了[Travis CI](https://travis-ci.org/) (travis-ci 用 GitHub账号登陆即可)

# 设置GitHub Personal Access Token
  1. 进入到 [GitHub Setting](https://github.com/settings/developers)页面
  2. 在左边菜单栏中选择 ["Personal access tokens"](https://github.com/settings/tokens)
  3. 点击右上角的 "Generate new token",进入“New personal access token”页面。
    ![](../../../../uploads/GitHub-token-1.png)
  4. 设置Token description（其实就是名称），选择相应的权限，如下图所示
    ![](../../../../uploads/GitHub-token-2.png)
  5. 点击"Generate token" 按钮,生成Personal access tokens
     **注意：这行token只会在刚刚创建完成后显示一次，以后不再显示**。 因此，复制并保存到本地
# 配置Travis CI
  ## 登录并配置Travis CI
    1. [Travis CI](https://travis-ci.org)是目前新兴的开源持续集成构建项目。可以直接使用GitHub账号登录
    2. 将鼠标放在用户名上，在弹出的菜单中点击“Accounts”，将会显示你在GitHub上的仓库。如下图所示
      ![](../../../../uploads/Travis-CI-1.png)
    3. 找到自己的博客项目，点击X号，将其变成√号。再点击右侧的齿轮，进入该仓库的配置页面
      ![](../../../../uploads/Travis-CI-2.png)
    4. 在项目的设置中开启 **Build only if .travis.yml is present** 这一项。如下图所示  
      ![](../../../../uploads/Travis-CI-3.png)
    好了，基本的配置已经完成了，下面即将开启 Travis CI之旅

  ## 本地安装Travis
      1. 首先安装Ruby，直接官网下载，双击安装就OK了
      2. 在Windows下，安装travis之前，需要解决一个问题：SSL证书问题，否则不能成功安装。详情请点击该链接：[参考教程](http://blog.csdn.net/chancein007/article/details/52940032)。
      3. 安装travis
        ```bash
        $ gem install travis
        # 或者可以指定travis版本
        $ gem install travis -v 1.8.8 --no-rdoc --no-ri
        ```
      **注意**  
      请确保你安装了ruby 2.3以上版本，否则 `gem install travis`会出错误
        ```bash
        $ ruby -v
        ```
        **MAC OS下可以使用 Homebrew来安装或者升级ruby**  
        ```bash
        $ brew install ruby

        # 如果本地已经存在比较老的ruby版本，请执行以下命令
        $ brew link --overwrite ruby

        $ ruby -v
        ```
  ## 创建并修改配置文件
    1. 打开博客项目文件夹，在项目根目录新建.travis.yml配置文件
      ```bash
      $ touch .travis.yml
      ```
    2. 本地登陆 travis
    > travis login 是走的是GitHub线路，所以，请放心使用

      ```bash
      $ travis login --debug
      $ Username: XXXXX
      $ Password for XXXXX: *******
      ```
    3. 执行下面的命令，加密上面生成的GitHub Personal access tokens，并添加到.travis.yml配置文件  
        ```bash
        # 这里的 REPO_TOKEN 是变量名,在后面的配置文件中会用到
        # TOKEN 是上面github生成的Token
        travis encrypt 'REPO_TOKEN=<TOKEN>' --add
        ```
        上述命令指向完后，.travis.yml配置文件的内容如下所示
        ```
        env:
          global:
            secure: soX2ds4ospOgwxNd4OSV.......
        ```
    4. 编辑 .travis.yml配置文件 (**注意** GitHub信息要换成你自已的)
      ```
      language: node_js
      node_js: node
        # - "6.9.4"  # nodejs的版本
      branches:
        only:
          - dev  # 设置自动化部署的源码分支
      cache:
        directories:
          - node_modules

      # ------------------------------------------------
      # 下面是Token加密信息
      # ------------------------------------------------
      env:
        global:
          secure: soX2ds4ospOgwxNd4OSV...........Akuok=

      before_install:
        - export TZ='Asia/Shanghai'
        # ------------------------------------------------
        # 设置github账户信息 注意修改成自己的信息
        # ------------------------------------------------
        - git config --global user.name "githubUsername"
        - git config --global user.email emailAddr@126.com


      # 安装依赖组件
      install:
        - npm install hexo-cli -g
        - npm install hexo --save
        - npm install
        - npm install hexo-generator-feed --save
        - npm install hexo-deployer-git --save
        - npm install hexo-renderer-ejs --save
        - npm install hexo-renderer-stylus --save
        - npm install hexo-renderer-marked --save
        - npm install hexo-generator-search --save

      before_script:
        - npm install -g gulp

      # 执行的命令
      script:
        - hexo clean
        - hexo generate
      # 执行的成功后执行
      after_success:
        # - hexo deploy
        - cd ./public
        - git init
        - git add --all .
        - git commit -m "Auto Builder From Travis"
        - git push --quiet --force https://$REPO_TOKEN@github.com/githubUsername/githubUsername.github.io.git master

      ```
