---
layout: page
title: '面向git编程'
categories: [tech]
tags: [original]
excerpt: 遵循良好的git使用习惯才能发挥git的最大效用，本文介绍了git的基本原则/对git友好的语法/提交的合理粒度/提交记录规范等等。
permalink: /en/:title/


---
> 作者: 李平海

## git使用基本原则

- 只包括需要共享/记录的文件，避免包括特大文件/无用的编译结果/特定系统文件/冗余文件 
- 使用对git友好的语法/项目结构/文件命名
- 在本地新建分支/特定分支开发新功能
- 变基/合并改动到合适的commit粒度
- 按格式写commit message
- 删除已合并的分支
- 按发布节点打tag


## 使用对git友好的语法/项目结构/文件命名

git变动以行/文件为单位，因此要避免无意义的行/文件改动，不在一行内写多个逻辑，避免在记录中难以辨别改了什么。

### 一行只写一个逻辑（赋值/判断）


#### 对象分开赋值，分行写html/css
```javascript
// incorrect
let a, b = 1, c = 2;

let a = b = c = 1;

// correct
let a; 
let b = 1; 
let c = 2;
```

```html
<!--incorrect-->
<div><span>标签1</span></div>

<!--correct-->
<div>
  <span>标签1</span>
</div>
```

```css
// incorrect
div{width: 100px; height:100px;}

// correct
div{
  width: 100px; 
  height:100px;
}
```


#### 对象属性/数组元素换行写


```js
// incorrect
let c = {foo: 1, bar: 2};

// correct
let c = {
  foo: 1, 
  bar: 2,
};

```

#### 避免冗长三元逻辑/三元分支赋值

```js
// incorrect
result ? success && success() : error && error('当前浏览器不支持复制操作，请手动复制!');

// correct
if (result) {
  success && success();
} else {
  error && error('当前浏览器不支持复制操作，请手动复制!');
}


// incorrect
  _SDKApp['load'].indexOf('clientGame') !== -1 ? pageInfo = '端游产品' : pageInfo = '页游产品';

// correct
if (_SDKApp['load'].indexOf('clientGame') !== -1){
  pageInfo = '端游产品';
} else {
  pageInfo = '页游产品'
}

```


#### 避免一行内同时赋值/执行方法和比较


```js
// incorrect
if (arrSource[i].split('=')[0].toLowerCase() === paramName) {
  //...
}

// correct
let sourceName = arrSource[i].split('=')[0].toLowerCase();
if(sourceName === paramName){
  //...
}
```

### 避免无意义的变动


#### 加上最后一行的逗号/分号


```javascript
// incorrect

let c = {
  foo: 1, 
  bar: 2
};

// correct

let c = {
  foo: 1, 
  bar: 2,
};
```
```css
// incorrect
  img {
    position: absolute;
    top: 0;
    height: 20px
  }

// correct
  img {
    position: absolute;
    top: 0;
    height: 20px;
  }
```

### 注意系统差异


#### 避免包括特定系统的文件


```
// .gitignore
.DS_Store
Thumbs.db
```

#### 文件名大小写？换行符？


```
// 全局设置不要自动改换行符
git config --global core.autocrlf false

// 全局设置大小写敏感
git config --global core.ignorecase false 

// 强制更新文件
git mv --force filename FILENAME
```

## 提交/分支/标签的粒度

提交commit是git使用的基本操作单位，控制好合适的提交粒度对git使用质量至关重要。

### 提交（commit）的粒度？

#### 提交应该是:

- 一个问题修复（fix） 
- 一个代码功能（特性）增加/删除（feat）
- 一个代码优化/重构（style，refactor）
- 一个文档改动（docs）
- 一次测试改动（test）
- 一个发布节点（release）
- 一次回滚（revert）
- 一次合并（merge）


#### 提交不应该是:

- 无意义的改动（空行/行尾符号/LForCRLF）
- 不完整的改动/跑不通的改动（一天的代码，一周的代码）
- 混杂了多个功能/改动


### 分支（branch）的意义

#### 仓库上的分支

- 一个历史版本 
- 一个特定版本
- 一个新功能特性



#### 本地的分支

- 开发新功能特性 

### 标签（tag/release）的意义

- 新的项目版本号 
- 一个公开发布/项目上线的版本节点


##	Git提交记录规范


### 按规范写提交记录的好处



- 使提交历史中的关键信息更分明
- 便于使用`git bisect`查找错误（忽略无关提交）
- 便于用脚本生成`CHANGELOG.md`


### Angular规范：



```md
type[(scope)]: subject
// 空一行
body（第一人称简要说明）
// 空一行
footer（不兼容变动/关闭issue）
```

以vue一次提交为例:


```
fix(ssr): render initial and used async css chunks (#7902)

compatibility with webpack 4 + mini CSS extraction

close #7897 
```

### 中文规范

对于非开源的业务项目，使用英文并不方便和直观，我们可以尝试用中文的提交规范

#### 中文提交格式

```md
类型[(影响范围)] 标题
// 空一行
简要说明
// 空一行
关联issue/不兼容提示
```

#### 提交类型



类型名称|含义
-|-
新增|新增api/特性
修复|修复问题
删除|删除api/参数等
更新|重构代码/优化代码/对代码中配置参数的变更/项目配置变化
优化|格式优化/文档更新/优化测试等
发布|新的版本号/release/tag(CHANGELOG在此改动)
回滚|回滚一次提交
合并|合并冲突/合并PR

#### 注意事项


- API变动需要明确指出API名字


中文规范例子



```
新增 Xx.xxx接口

此接口功能很强大，传参请随意，不兼容IE8以下

close #1
```

##	Git提交记录操作

使用git出错或者采用临时commit在所难免，我们需要对commit进行合理优化


### 修改最近提交信息


```
git commit --amend
```

### 修改多次提交


```
git rebase -i CommitHash
```

### 移动提交记录	


```
git rebase --onto TargetBranch/TargetCommit 
```

详细操作请参阅[Git变基](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA)。

## 参考资料

[Linux提交记录](https://github.com/torvalds/linux/commits/master)

[vue提交记录](https://github.com/vuejs/vue/commits/)

[Angular提交规范](https://gist.github.com/stephenparish/9941e89d80e2bc58a153)

[阮老师谈commit message](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)

[git demo](http://gitlab.game.yypm.com:10080/lipinghai/git-demo)

[conventional-changelog-yygame规范](https://github.com/LiPinghai/conventional-changelog-yygame)