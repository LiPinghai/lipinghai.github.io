---
layout: page
title: '如何用规范的git提交记录生成CHANGELOG?'
categories: [tech]
tags: [original]
excerpt: 知名前端web项目AngularJS的提交记录在业内被许多人称赞，其规范同时也逐渐被大家引用。但日常开发中不一定方便都用英文撰写commit message，这里介绍一个兼容中文的规范集——[conventional-changelog-common](https://github.com/LiPinghai/conventional-changelog-common)。
permalink: /en/:title/


---
> 作者: 李平海

知名前端web项目AngularJS的提交记录在业内被许多人称赞，其[规范](https://www.npmjs.com/package/conventional-changelog-angular)同时也逐渐被大家引用。但日常开发中不一定方便都用英文撰写commit message，这里介绍一个兼容中文的规范集——[conventional-changelog-common](https://github.com/LiPinghai/conventional-changelog-common)。

#### commit message类型

中文类型|英文类型|含义
-|-|-
新增|feat|新增api/功能
修复|fix|修复问题
删除|delete|删除api/参数等破坏性改动
更新|refactor|重构代码/优化代码/对代码中配置参数的变更/项目配置变化
优化|perf/chore/docs|格式优化/文档更新/优化测试等
发布|release|新的版本号/release/tag(CHANGELOG在此改动)
回滚|revert|回滚一次提交
合并|merge|合并冲突/合并PR

#### commit message提交格式

```md
类型[(影响范围)] 标题
// 空一行
简要说明
// 空一行
关联issue/不兼容提示
```
例子：
```
新增 xx.xxx接口

此接口用于xxxxx，传参xxx，不兼容IE8以下

close #1
```

## 如何集成到前端项目中？

安装*conventional-changelog-cli*和*conventional-changelogc-common*依赖。

```shell
npm i conventional-changelog-cli conventional-changelog-common -D
```

## 如何生成changelog？

在package.json加上以下脚本(详细参数请运行*npx conventional-changelog-cli --help*查看)：
```json
{
    "scripts": {
        "changelog": "conventional-changelog -i CHANGELOG.md -s -r 1 -n ./node_modules/conventional-changelog-common" 
    }
}
```
再运行
```shell
npm run changelog
```
即在项目根目录生成/追加*CHANGELOG.md*

CHANGELOG效果如下：

--------
<a name="0.3.30"></a>
## [0.3.30](https://github.com/LiPinghai/conventional-changelog-common) (2018-12-19)

### 更新:

* 更新 用xxx替换所有xx ([d3813ff](https://github.com/LiPinghai/conventional-changelog-common))
* 更新 优化xxx接口参数 ([055d7c1](https://github.com/LiPinghai/conventional-changelog-common))
* 更新 改xx.xxx()监听xxx为监听xxx ([bb7a489](https://github.com/LiPinghai/conventional-changelog-common))

### 修复:

* 修复 xx和xx模块埋点覆盖冲突问题 ([1894a3a](https://github.com/LiPinghai/conventional-changelog-common))
* 修复 在xx环境下，xx报错的问题 ([e577220](https://github.com/LiPinghai/conventional-changelog-common))

### 新增:

* 新增 xx增加xx接口 ([fb40edc](https://github.com/LiPinghai/conventional-changelog-common))
* 新增 xx模块 ([fe1d1bd](https://github.com/LiPinghai/conventional-changelog-common))
--------