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
const express = require("express");
const cors = require("cors");
const { Configuration, OpenAIApi } = require("openai");
require("dotenv").config();

const app = express();

// Middleware
app.use(cors());
app.use(express.json());

// Configure OpenAI API
const configuration = new Configuration({
  apiKey: process.env.OPENAI_API_KEY,
});
const openai = new OpenAIApi(configuration);

// Define the chat endpoint
app.post("/chatbot/chat", async (req, res) => {
  const { message } = req.body;

  try {
    const response = await openai.createChatCompletion({
      model: "gpt-3.5-turbo",
      messages: [{ role: "user", content: message }],
    });

    res.json({ reply: response.data.choices[0].message.content });
  } catch (error) {
    console.error(error);
    res.status(500).json({ error: "Something went wrong with the API." });
  }
});

// Start the server
const PORT = process.env.PORT || 3005;
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

现在我们可以运行服务：

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
curl -X POST http://localhost:3005/chatbot/chat \
-H "Content-Type: application/json" \
-d '{"message": "Hello, ChatGPT!"}'
```

如果一切正常，将收到来自ChatGPT的响应。

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
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'
import App from './App.jsx'

createRoot(document.getElementById('root')).render(
  <StrictMode>
      <App />
  </StrictMode>,
```

#### 在src/App.jsx中构建聊天界面
将App.jsx的内容替换为以下简单的聊天机器人UI：

```javascript
import * as React from 'react';
import { useState } from "react";
import './App.css'
import axios from "axios";

const Default = () => {

    const [inputText, setInputText] = useState("");

    const list = []
    const [chatList, setChatList] = useState(list);

    const addChatItem = (newItem) => {
      setChatList(prevItems => [...prevItems, newItem]);
    };

    const handleKeyDown = (event) => {
      if (event.key === 'Enter') {
        sendMessage();
      }
    };

    const sendMessage = async() => {

        const input = document.querySelector('.chat-text');
        const question = input.value;
        addChatItem({from:'self', content: question});
        setInputText('');
        
        const response = await axios.post("/chatbot/chat", {
          question,
        });
        addChatItem({from:'other', content: response.data});
 
    };

    return (
      <div className='control-pane'>
        <div className="control-section">
          <h1 hidden={chatList.length}>What can I help with?</h1>
          <ul className='chat-list' hidden={!chatList.length}>
            {chatList.map((chat, index) => (
              <li className='chat-item' key={index}>
                <div className={'chat-content ' + (chat.from === 'self' ? 'chat-content-self': 'chat-content-other')  }>
                  {chat.content}
                </div>
              </li>
            ))}
          </ul>
          <div className={'chat-input '+ (chatList.length ? 'chat-input-chated' : '' )}>
            <input maxLength="500" rows="5" value={inputText} className='chat-text' type="text" onChange={(e)=>setInputText(e.target.value)} onKeyDown={handleKeyDown} placeholder="Ask me anything"/>
            <button className={'submit-button ' + (inputText ? 'inputed' : '')}  onClick={sendMessage}>↑</button>
          </div>
        </div>
      </div>);
};

export default Default;
```

### 第三步：在src/App.css中添加样式
为聊天机器人UI创建基本样式：

```css
#root {
  max-width: 1280px;
  min-width: 500px;
  margin: 0 auto;
  text-align: center;
  height: 100vh;
  position: relative;
  box-sizing: border-box;
}

.chat-input {
  color: rgb(13, 13, 13);
  background-color: rgb(244, 244, 244);
  line-height: 40px;
  padding: 6px 10px;
  border-radius: 25px;
  width: 100%;
  box-sizing: border-box;
}

.chat-input-chated {
  position: absolute;
  bottom: 10px;
}

.chat-text{
  background-color: rgb(244, 244, 244);
  border: none;
  font-size: 16px;
  width: 85%;
}

.submit-button {
  display: flex;
  justify-content: center;
  background-color: rgb(227, 227, 227);
  color: rgb(244, 244, 244);
  font-size: 28px;
  border-radius: 100%;
  border: none;
  width: 32px;
  height: 32px;
  font-family: system-ui;
  font-weight: 600;
  cursor: pointer;
  float: right;
  top: 4px;
  position: relative;

}

.inputed {
  background-color: rgb(0, 0, 0);
}

ul {
  padding-left: 0;
}

li {
  list-style: none;
}

.chat-list {
  padding-left: 0;
  max-height: calc(100vh - 80px);
  overflow: auto;
  scroll-behavior: smooth;
}

.chat-content {
  width: fit-content;
  max-width: 400px;
  display: flex;
  margin-bottom: 8px;
  padding: 10px 20px;
  line-height: 22px;
  border-radius: 25px;
  word-break: break-word;
  text-align: left;
}

.chat-content-self {
  color: rgb(13, 13, 13);
  background-color: rgb(244, 244, 244);
  justify-self: end;
}

input:focus{
  outline: none;
}
```

### 第四步：运行和测试
启动Vite开发服务：

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

启动Express服务：

```bash
node server.js
```

将生成的dist目录中的文件部署到静态托管服务（如Vercel、Netlify）或您自定义的服务器。
