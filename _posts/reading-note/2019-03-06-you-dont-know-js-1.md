---
layout: default
title: 《你不知道的JS》上卷读书笔记
categories: [tech]
tags: [reading-note]
excerpt: 《你不知道的JS》上卷读书笔记。Kyle Simpson的这部*you don't know JavaScript*实属不可多得的好书，深入浅出地解释了语言本身的诸多令人费解复杂概念的内部实现，读罢如醍醐灌顶豁然开朗，重建了对JS的整体视野，建议有一年以上经验的前端工程师都读读。
---
# 《你不知道的JS》上卷读书笔记
> Kyle Simpson的这部*you don't know JavaScript*实属不可多得的好书，深入浅出地解释了语言本身的诸多令人费解复杂概念的内部实现，读罢如醍醐灌顶豁然开朗，重建了对JS的整体视野，建议有一年以上经验的前端工程师都读读。

## 作用域

### js是一门编译语言
- 分词/词法分析（tokenizing/lexing）
- 解析/语法分析（parsing），生成ast
- 可执行代码生成

### 声明和赋值   
- 变量的赋值操作
    1. LHS,声明变量
    2. 赋值
- LHS: 查找操作目标，如*var b = a*中的b，非严格模式下不成功则创建该变量，严格模式下ReferenceError
- RHS: 查找目标的值，如*var b = a*中的a, 不成功ReferenceError

### 词法作用域
- js采用*词法作用域*（区别于*bash脚本等动态作用域*）
- eval with欺骗词法，导致解析时无法优化，性能下降
- 作用域嵌套
    - 全局作用域
    - 函数作用域
        - 匿名与具名（匿名难以调试，难以自调用，可读性差）
        - 立即执行函数表达式（IIFE, Immediately Invoked Function Expression）
    - 块作用域
        - try catch也是块级作用域（babel转译到es3/5）

### 变量提升
 - 因为在编译阶段先声明，运行阶段再赋值，所以出现变量提升。
 - 函数提升比变量提升优先
 - 多个相同var声明或函数声明，前面的会被忽略

### 作用域闭包
- 闭包即内部函数在所在的*词法作用域*之外被调用
- 常见闭包使用
    - setTimeout,setInterval
    - 事件监听
    - ajax
    - postMessage
    - web worker
    - 其他异步或同步方法中用到回调函数的情况

## this

### 对this的错误认识
- this指向自身
- this指向函数作用域

### this是什么
- this即函数的调用者（优先级升序排列）
    - 默认绑定（到全局）
    - 隐式绑定（函数父级）
    - 显式绑定（call，apply，forEach等内置函数）
    - new调用绑定
        - 创建一个全新对象
        - 新对象执行原型链接
        - 新对象绑定到函数调用的*this*
        - 如果函数没有其他返回，则返回此新对象  
- call函数假如传入了原始值（数字字符串布尔），会转换成对象形式（即new Number(...)）

## 对象

### 内置对象
- String
- Number
- Boolean
- Object
- Symbol
- Function
- Array
- Date
- RegExp
- Error
 
### 对象的知识点
- 字符串/数值/布尔值等在必要时会自动转成对象，比如42.1.toFixed(0)
- 为什么typyof null === 'object'?
    - js中二进制前三位都为0则会被判定为object，null全部为0，这是语言本身的bug
- 属性名永远是字符串，不是字符串会自动转成字符串
    - 但es6中的*Map*允许其他类型的属性名，书中说也会转成字符串，存疑。
- 数组也可以加属性和方法，此时数组的length不变
- 复制对象
    - JSON安全的对象，JSON.parse(JSON.stringify(obj))
    - Object.assign({}, obj)

- 属性描述符
    - writabe,属性是否可修改
    - configurable,属性的描述符是否可改（false不可逆）
    - enumeralbe,是否可枚举
- 如何让对象不可改？
    - writabe: false, configurable: false
    - Object.preventExtension() 禁止增加新属性
    - Object.seal(),密封对象,即Object.preventExtension() + 所有属性configurable: false
    - Object.freeze(),冻结对象,即Object.seal() + 所有属性writabe: false
- 如何区分属性是undefined还是属性不存在？
    - 'a' in obj
    - obj.hasOwnProperty()
- 如何判断属性可枚举？
    - 'a' in obj
    - obj.propertyIsEnumable()

## 混合对象和“类”

- js没有真正的类，类和实例是抽象和具象的关系，而js的“类”实现是基于原型链，只存在关联，不存在继承和实例化。

## 原型
- 所有普通的原型链最终都指向内置的Object.prototype
- js只有对象，没有类，是真正的“面向对象”语言
- js中有一种奇怪的行为一直在被无耻地滥用，那就是模仿类。

- 修改对象的原型 Object.setPrototypeOf(ojb, pObj)
- 检测原型 obj.prototype.isPrototypeOf(x)
- 获取原型 Object.getPrototype(obj)

## 行为委托

- 委托比“类”要简洁
- Object.create()语法优于class
- ES6中的class只是语法糖，依然是基于原型


## 勘误
### p57 es6模块写法
第一处错误：
```
bar.js
    function hello(who){
        return "Let me intropduce:" + who;
    }    
    export hello;

foo.js
    // 仅从bar模块导入hello()
    import hello from "bar";
    // 其他代码
```
这里写法是错的，直接import时，引入的是目标模块中export default的内容，浏览器会报错*Uncaught SyntaxError: The requested module 'xx.js' does not provide an export named 'default'*。

用对象解构写法*import {hello} from "bar"*,此时bar.js应*export {hello}*,或直接*export function hello...*

三种正确写法：
```js
bar.js
    function hello(who){
        return "Let me intropduce:" + who;
    }    
    export {hello};

foo.js
    import {hello} from "bar";
```
```js
bar.js
    export function hello(who){
        return "Let me intropduce:" + who;
    }
    
foo.js
    import {hello} from "bar";
```
```js
bar.js
    export default function hello(who){
        return "Let me intropduce:" + who;
    }
    
foo.js
    import hello from "bar";
```
第二处错误：
```js
module foo from "foo";
```
```js
module foo from "foo";
```
module关键字用法，实际不存在

### p80 this
```js
function foo() {
  var a = 2;
  this.bar();
}
function bar() {
  console.log(this.a);
}
foo(); // ReferenceError: a is not defined
```
实际上不会报错，而是输出*undefined*

### p106 对象的属性名

书中说对象的属性名永远是字符串，但es6中的*Map*允许其他类型的属性名，书中说也会转成字符串，存疑。