---
title: Solidity 入门
copyright: true
date: 2018-03-22 11:47:25
tags: 深入浅出区块链
categories: Solidity
---

智能合约本质上是一段程序，程序是需要用编程语言来实现的。和以太坊客户端一样，智能合约也有很多语言版本，这里使用的是官方推荐的编程语言Solidity，文件扩展名以.sol结尾。下面简单简单介绍下Solidity的语法。


# 语法
## 编译器版本指定
和其他语言一样，Solidity语言也是在不断的发展和改进的，不同的版本支持的功能不同，所以sol文件需要指定版本号，通常在sol文件的第一行需要指定。语法如下：
```
pragma solidity ^0.4.4;
```
上面的意思这个sol文件需要在0.4.4之后的版本上运行，其中的“^”符号表示不支持0.5.0及之后的版本。

## 注释
在Solidity中使用“//”表示单行注释，使用“/ ... / ”表示多行注释
```
// 这是但行注释

/*
这是
多行注释
*/
```

## 变量声明和常见数据类型
```
bool b = false; // 布尔类型，默认值为false
uint i = 0;     // 整型
address addr;   // 地址类型，这是以太坊中的一个特殊类型，为20个字节的值，用来保存一个以太坊地址
byte32 by;      //

bytes memory varBy;  // 字节数组
string memory str;   // UTF-8字符数组
uint[] memory arr;   // 整型数组

mapping(address => uint) public balances;   // 映射，相当于一个Hash表
```

## 枚举
```
enum Color{RED, GREEN, YELLOW};   // 默认从0开始
Color light;
light.RED;    // 0
light.GREEN;  // 1
light.YELLOW; // 2
```

## 结构体
```
// 定义一个结构体，包含地址和数量两个属性
struct Player {
        address addr;
        uint amount;
    }
```

## 函数
Solidity中函数的定义语法如下:
```
function f(<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)] {

 // function body
}
```
其中`<parameter types>`指函数的参数及类型:
* `{internal|external}`这两个关键字规定了函数的调用方式，`internal`指内部调用，能直接使用上下文环境中的数据; `external`实现为合约的外部消息调用. 默认是`internal`.
* `[pure|constant|view|payable]` 这四个关键字用来说明函数属性。`pure`关键字来源于函数式编程，表明这个函数体是一个纯函数计算不能调用其他函数；`cosntant`关键字在0.4.17版本后将 **废弃** 使用；`view`关键字表明这个函数是只读的不能修改状态；如果一个函数需要进行货币操作，**必须** 要带上`payable`关键字。
* `[returns (<return types>)]` 用来指明函数的返回类型

以上是Solidity语法的简单介绍，详细内容可参看官方教程(http://solidity-cn.readthedocs.io/zh/latest/).

# 编译和执行
智能合约在以太坊上运行，需要进行编译和部署。这里推荐使用[Truffle](http://truffleframework.com/docs/getting_started/contracts)工具。Truffle是针对基于以太坊的[Solidity](http://solidity-cn.readthedocs.io/zh/latest/)语言的一套开发框架。本身基于Javascript。它集成了智能合约的开发，测试，部署，以及一个交互式的命令行功能，极大的方便了调试开发。Truffle的安装命令如下：
```bash
$ npm install -g truffle
```
安装完成后使用`truffle init`命令进行初始化。
```bash
$ truffle init               
Downloading...
Unpacking...
Setting up...
Unbox successful. Sweet!

Commands:

  Compile:        truffle compile
  Migrate:        truffle migrate
  Test contracts: truffle test
```
truffle会自动下载一个空的项目工程并提供编译、部署、测试三个命令工具。

项目初始化后目录结构如下：
```bash
.
├── contracts               
│   └── Migrations.sol
├── migrations
│   └── 1_initial_migration.js
├── test
├── truffle-config.js
└── truffle.js
```
其中 **contracts** 文件夹是用来存放智能合约的地方；

**migrations** 文件夹用来实现部署智能合约的功能；

**test** 文件夹用来存放合约的测试文件；

**truffle.js** 默认配置文件

**truffle-config.js** Windows下默认配置文件名与truffle冲突，可使用该文件解决

项目初始化后需要修改配置文件，本文中使用了Ganache, 设置为本地的8545端口，修改truffle.js文件如下：
```
module.exports = {
  networks: {
    development: {
      host: "127.0.0.1",
      port: 8545,
      network_id: "*" // Match any network id
    }
  }
};
```
设置完成后就可以开始实现智能合约了。

一个简单的HelloWord智能合约大致如下：
```
pragma solidity ^0.4.4;


contract HelloWorld {
    function renderHelloWorld() public pure returns (string) {
        return "Hello World";
    }
}
```
上面实现了一个输出“Hello World”的智能合约。在contracts文件夹中新建一个HelloWorld.sol文件，并将上面内容保存到这个文件中。保存完成后目录结构如下:
```
.
├── contracts
│   ├── HelloWorld.sol
│   └── Migrations.sol
├── migrations
│   └── 1_initial_migration.js
├── test
├── truffle-config.js
└── truffle.js
```
然后用truffle进行编译。
```bash
$ truffle compile
Compiling ./contracts/HelloWorld.sol...
Compiling ./contracts/Migrations.sol...
Writing artifacts to ./build/contracts
```
编译成功后会当前目录的build文件夹下生成新的文件。下一步就是将智能合约部署到以太坊网络上，在migrations文件夹下新建一个，内容如下：
```
var HelloWorld = artifacts.require("HelloWorld");  // 获取HelloWorld合约
module.exports = function(deployer) {
    deployer.deploy(HelloWorld);                  // 部署到以太坊上
};
```
保存后当前目录结构如下：
```
.
├── build
│   └── contracts
│       ├── HelloWorld.json
│       └── Migrations.json
├── contracts
│   ├── HelloWorld.sol
│   └── Migrations.sol
├── migrations
│   ├── 1_initial_migration.js
│   └── 2_deploy_contracts.js
├── test
├── truffle-config.js
└── truffle.js
```
使用truffle migrate命令进行部署。
```bash
$ truffle migrate       

Using network 'development'.

Running migration: 1_initial_migration.js
  Deploying Migrations...
  ... 0x023e8ae8837ea28c9672f2adfba4f8a693bdb0483c4dd44bc69946e8f2a33b36
  Migrations: 0x45482a119882930486c0dd210dff81e0eb451fa2
Saving successful migration to network...
  ... 0xec903ccaee280965b6ec3172df382efb614f798ae31c66a167554e02191d3000
Saving artifacts...
Running migration: 2_deploy_contracts.js
  Deploying HelloWorld...
  ... 0x6f6e5e213cf109d6780eca1d687b8cd04efcc4ce4c7682c2c1e84a7be4f8b4da
  HelloWorld: 0x5878837601cb2d5da7190c4c42f6a5399ca96784
Saving successful migration to network...
  ... 0xf815aba07df8a2e9981ea2360c3f37abf01d6ec61059329aa8a4d36b912fc5c5
Saving artifacts...
```
到这里，智能合约这部分已经完成了，接下来是给智能合约做个UI，实现一个DApp。

在当前目录下新建一个app的文件夹，然后在该文件夹中创建index.html,app.js 这两个文件, 再把前面编译生成的文件HelloWorld.json拷贝到这里(build目录下）。另外需要下载几个js库，一个是常用的jquery.js, 一个是用来与以太坊节点交互的web3.js(它通过RPC的方式与节点进行通信)，还有一个是truffle-contract.js, 它是对智能合约的js封装。app的目录结构如下：
```bash
.
├── HelloWorld.json
├── app.js
├── index.html
└── js
    ├── jquery.min.js
    ├── truffle-contract.js
    └── web3.min.js
```
在index.html中实现了一个简单文本块，并将需要的js文件引用进来，内容如下:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Demo</title>
</head>
<body>
    <div style="text-align:center;font-size:50px" id='content'>
        加载中...
    </div>
</body>
<script src="js/jquery.min.js"></script>
<script src="js/web3.min.js"></script>
<script src="js/truffle-contract.js"></script>
<script src="app.js"></script>
</html>
```

在app.js中，将会加载HelloWorld智能合约，加载后调用合约中的函数并修改网页显示，文件内容如下：
```javascript
$(function() {
  $(window).load(function() {
      // 初始化web3，使用本地的8545端口
      var web3Provider = new Web3.providers.HttpProvider('http://localhost:8545');

      // 获取智能合约的ABI（Application Binary Interface）文件
      $.getJSON('HelloWorld.json', function(data){
          var HelloWorldArtifact = data;

          // 初始化智能合约
          HelloWorldContract = TruffleContract(HelloWorldArtifact);
          HelloWorldContract.setProvider(web3Provider);

          // 通过默认的合约地址获取实例
          HelloWorldContract.deployed()
          .then(function(instance){

              // 通过获取到实例调用函数，这里函数返回的是一个promise对象
              instance.renderHelloWorld().then(function(result){
                  // 更新页面内容
                  // $("#content").text(result);
              })
         }).catch(function(err){
                console.log(err.message);
         })

      })
  });
});
```

以上DApp基本实现完成，然后是它的启动，这里是lite-server来启动。
1. 初始化一个package.json
  ```
  npm init
  ```
2. 更新package.json内容如下：
  ```javascript
  {
    "name": "pet-shop",
    "version": "1.0.0",
    "description": "",
    "main": "truffle.js",
    "directories": {
      "test": "test"
    },
    "scripts": {
      "dev": "lite-server",
      "test": "echo \"Error: no test specified\" && exit 1"
    },
    "author": "",
    "license": "ISC",
    "devDependencies": {
      "lite-server": "^2.3.0"
    }
  }
  ```
3. 安装lite-server
  ```
  npm install
  ```
4. 启动
  ```
  npm run dev
  ```
打开浏览器，访问localhost:3000就可以看到如下效果。  
