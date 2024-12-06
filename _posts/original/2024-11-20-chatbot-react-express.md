
---
layout: page
title: '构建全栈聊天机器人：React前端、Express后端以及OpenAI API集成'
categories: [tech]
tags: [original]
excerpt: '用React、Express和OpenAI API构建聊天机器人。'
---

聊天机器人在构建互动型网络应用中越来越受欢迎，结合React、Express和OpenAI API的力量，使开发变得简单。在本教程中，您将学习如何构建一个功能齐全的聊天机器人网站，包括：

* 使用React前端构建一个类似ChatGPT的用户友好界面。
* 使用Express后端处理API调用并作为静态文件服务React应用。
* 集成OpenAI API，实现自然语言处理。

在本教程结束时，您将拥有一个集成无缝互动和先进AI能力的聊天机器人网站，可以直接部署。让我们开始吧！

## 构建服务器

### 1. 项目设置

我们使用Node.js和Express来构建一个简单易用的API。

1. 创建一个新的项目目录：

```bash
  mkdir chatbot
  cd chatbot
```

2. 初始化Node.js项目：

```bash
npm init -y
```

此时将生成一个package.json文件。

3. 安装所需的依赖：

```bash
npm install express cors openai dotenv
```

### 2. 获取OpenAI API密钥

1. 如果没有账户，请访问[OpenAI](https://platform.openai.com/)注册。
2. 从[OpenAI API密钥页面](https://platform.openai.com/api-keys)生成一个API密钥。
3. 将API密钥保密；稍后将在.env文件中使用。

### 3. 创建Express服务器

在项目根目录创建一个名为server.js的文件。

在文件中添加以下代码以构建服务器：

```javascript
...（此处省略，保留代码不翻译）
```

现在我们可以运行服务器：

```bash
node server.js
```

您还可以在package.json中添加脚本：
```js
  "scripts": {
    "server": "node server.js",
  }
```
然后可以通过命令`npm run server`启动服务器。

接下来，可以使用Postman或curl工具测试`/chat`端点：

```bash
...（此处省略，保留代码不翻译）
```

如果一切正常，您将收到来自ChatGPT的响应。

## 创建React Web应用

### 第一步：设置React项目
创建一个新的React + Vite项目：

```bash
npm create vite@latest chatbot-frontend
```

选择以下选项：

* 框架：React
* 变体：JavaScript或TypeScript（根据您偏好）

进入项目目录并安装依赖：

```bash
cd chatbot-frontend
npm install
```

启动开发服务器：

```bash
npm run dev
```

这将在http://localhost:5173启动应用。

### 第二步：创建聊天机器人页面

#### 修改src/main.jsx
此文件是应用的入口点，应如下所示：

```javascript
...（此处省略，保留代码不翻译）
```

#### 在src/App.jsx中构建聊天界面
将App.jsx的内容替换为以下简单的聊天机器人UI：

```javascript
...（此处省略，保留代码不翻译）
```

### 第三步：在src/App.css中添加样式
为聊天机器人UI创建基本样式：

```css
...（此处省略，保留代码不翻译）
```

### 第四步：运行和测试
启动Vite开发服务器：

```bash
npm run dev
```

在浏览器中打开应用：http://localhost:5173

在输入框中输入消息并发送。聊天机器人将调用后端并显示ChatGPT的响应。

## 构建和部署
构建生产环境应用：

```bash
npm run build
```

这将在客户端生成React应用的生产版本。

返回项目根目录：

```bash
cd ..
```

启动Express服务器：

```bash
node server.js
```

将生成的dist目录中的文件部署到静态托管服务（如Vercel、Netlify）或您自定义的服务器。
