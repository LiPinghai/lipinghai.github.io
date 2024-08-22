---
layout: page
title: 'npm script中文文档'
categories: [tech]
tags: [translation]
excerpt: npm script可谓前端日常用得最多的命令。本文译自npm官方script文档，详尽解释npm script用法。
permalink: /zh/:title/
---
> 译者: 李平海

本文翻译自[npm官方script文档](https://docs.npmjs.com/misc/scripts)

npm 支持运行package.json里*scripts*属性中的脚本，包括：

* `prepublish`：在`npm publish`命令之前运行（也会在不带参数的`npm install`命令前运行，详情在下段描述）
* `prepare`: 在两种情况前运行，一是`npm publish`命令前，二是不带参数的`npm install`命令；它会在`prepublish`之后、`prepublishOnly`之前执行
* `prepublishOnly`:  在`npm publish`命令前执行
* `publish,postpublish`： 在`npm publish`命令后执行
* `preinstall`: 在`npm install`命令前执行
* `install,postinstall`： 在`npm install`命令后执行
* `preuninstall，uninstall`: 在`npm uninstall`命令前执行 
* `postuninstall` ： 在`npm uninstall`命令后执行
* `preversion`：在改变包的version前执行
* `version`： 在改变包的version后，但提交之前执行
* `postversion`： 在提交version变更后执行
* `pretest， test， posttest`： 伴随`npm test`命令
* `prestop， stop， poststop`： 伴随`npm stop`命令
* `restart, start, poststart`: 伴随 `npm start`命令
* `pre restart， restart， poststart`： 伴随 `npm restart`命令。提示：假如scripts里没有写restart命令，npm默认会运行start、stop
* `preshrinkwrap, shrinkwrap, postshrinkwrap`: 伴随 `npm shrinkwrap` 命令（用于固定依赖包版本）

另外，所有的script都可以用 `npm run-script <stage>`命令执行, pre 和 post 会自动匹配对应的script（例如 `premyscript，my script， postmyscript`），依赖包中的script可以用`npm explore <pkg> -- npm run <stage>`  来执行

## prepublish 和 prepare

### 废弃告示：

从npm@1.1.71开始，npm CLI 会在`npm publish`和`npm install`之前执行`prepubish`脚本，因为这做法便于准备package环境（几种用法如本段下面描述）。但事实证明这让人非常困惑。所以在npm@4.0.0后，新增来一种事件`prepare`来替代上述功能。另外一种新事件`prepublishOnly`作为替代策略，让用户避免以往npm版本的混乱行为。`prepublishOnly` 只在`npm publish`前执行（例如，publish前最后执行一次测试，以确保无误）

*重要提示*：在npm5中， `prepublish`只在`npm publish`前执行，即取代`prepublishOnly`，所以npm6即以后的版本会放弃`prepublishOnly`。到时就忘了这些矬事吧。

 [https://github.com/npm/npm/issues/10074](https://github.com/npm/npm/issues/10074) ，这里对此变更有过漫长的争论可作参考。

## 用法：

如果你需要在使用package之前执行某些操作，而不依赖于目标系统的操作系统或目录结构，请使用prepublish脚本。例如以下任务：

* 把coffeescript源码编译成JS
* 压缩js源码
* 获取你的package所需的远程资源

在publish时做这些事情的好处是，它们可以在同一处完成，从而降低复杂性和差异性。另外，这意味着：

* 你可以依赖于coffee－script作为devDependency，但你的用户不需要安装它。
* 你不需要在包中包含min版本，以节省空间
* 你的用户的电脑上不必具有curl或wget或其他系统工具

## 预设脚本

npm 预设了几个script内容：

```javascript
"start": "node server.js"
```

如果你的package根目录中有server.js, npm会默认以之作为start命令执行。

```javascript
"install": "node-gyp rebuild"
```

如果你的package根目录下中有binding.gyp而你又没定义 install或preinstall脚本，npm会默认以npm－gyp编译。

## 用户

如果使用root权限调用npm，那么它会将uid更改为用户配置指定的uid，默认为nobody。设置unsafe-perm以使用root权限运行脚本。

## 环境

npm 脚本运行在 npm的设置和进程的当前状态 的环境中。(不知道怎么翻)
Package scripts run in an environment where many pieces of information are made available regarding the setup of npm and the current state of the process.

### 路径

如果你依赖的模块（如测试模块）定义了可执行脚本，那么这些可执行文件将被添加到PATH中执行。所以，如果你的package.json有这样的依赖：

```javascript
{ 
  "name" : "foo", 
  "dependencies" : { "bar" : "0.1.x" }, 
  "scripts": { "start" : "bar ./test" } 
}
```
那么在你`npm install`时，bar会导进`node_modules/.bin`目录中,你可以运行`npm start`执行 bar脚本。

### package.json变量

package.json的属性名会拼接上`npm_package_`前缀。例如如果你的package.json文件中有`{“name”：“foo”，“version”：“1.2.5”}）`，那么你的脚本中`npm_package_name`会是`foo`，而`npm_package_version`会是`1.2.5`

### 配置

使用`npm_config_`前缀将配置参数放在环境中。例如你可以通过检查`npm_config_root`环境变量来查看目前生效的root配置。

*特殊*：package.json中的config对象

如果npm config中存在<name> [@ <version>]：<key>的配置参数，则package.json里config对应的key将在环境中被覆盖。例如，如果package.json中：

```javascript
{ 
  "name" : "foo", 
  "config" : { "port" : "8080" }, 
  "scripts" : { 
    "start" : "node server.js" 
  } 
}
```
而server.js中这么写：
```javascript
http.createServer(...).listen(process.env.npm_package_config_port）
```
那么用户可以这样覆写port变量：
```javascript
npm config set foo:port 80
```

### 当前的生命周期事件

最后，`npm_lifecycle_event`环境变量代表当前执行循环的哪个阶段。所以，你可以使用单个脚本用于根据当前正在发生的事件切换的进程的不同部分。

对象按照层级以下划线连接，如果您的package.json中有
```javascript
{“scripts”：{“install”：“foo.js”}}
```
则可以在脚本中看到：
```javascript
process.env.npm_package_scripts_install === "foo.js"
```

## 例子
假如你的package.json有如下设置：
```javascript
{ "scripts" :
  { 
    "install" : "scripts/install.js", 
    "postinstall" : "scripts/install.js", 
    "uninstall" : "scripts/uninstall.js"
  }
}
```

那么在生命周期的`install`和`postinstall`阶段将调用`scripts/install.js`，在`uninstall`时将会调用`scripts/uninstall.js`。
由于`scripts/install.js`在两个不同的阶段都有运行，因此在这种情况下，查看`npm_lifecycle_event`变量是聪明的做法。

如果你想要运行make命令也没问题，这样写：

```javascript
{ "scripts" :
  { 
    "preinstall" : "./configure", 
    "install" : "make && make install", 
    "test" : "make test"
  }
}
```

## 退出脚本

npm脚本是通过将该行作为脚本参数传给sh来运行的。

如果脚本不是以退出码0退出，则会中止该进程。

请注意，这些脚本文件不必是nodejs甚至是javascript程序。他们只需要一些可执行文件。

## 钩子脚本

如果要为*所有package*在某生命周期事件中运行某脚本，则可以使用钩子脚本。

在`node_modules/.hooks/{eventname}`中放置一个可执行文件，所有装在根目录下的package在运行到该生命周期节点时都会执行。

Hook脚本的运行方式与package.json里的脚本完全相同。也就是说，它们与上述env在一个单独的子进程中。

## 实践建议

* 不要以0以外的错误码退出，除非你真的想这样做。因为这将导致npm操作失败(`install`脚本除外)，并可能会回滚。如果这个错误无关紧要或只会阻断一些小功能，那么最好只是输出一个警告，让线程成功退出。

* 尽量不要使用npm脚本来做npm本身就可以做的事情。通读package.json文件，你可以学到所有通过简单、合适的配置来具体描述你的package。这通常更健壮和通用。

* 通过检查`env`来决定把东西装在哪。例如，如果`npm_config_binroot`环境变量设置为`/home/user/bin`，那么不要尝试将可执行文件安装到`/usr/local/bin`中。用户可能因为某种原因设置了这种方式。

* 不要在脚本命令里加`sudo`前缀。如果由于某些原因需要root权限，否则会报错；那么用户应该考虑以sudo运行npm命令。

* 不要使用`install`脚本。使用`.gyp`文件进行编译，其他情况则使用`prepublish`。你最好永远不要自定义`preinstall`或`install`脚本。如果你要这样做，请优先考虑是否有其他办法。`install`或`preinstall`脚本的唯一正确使用场景是用作设置那些编译的目标目录。
