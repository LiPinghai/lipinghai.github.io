---
layout: page
title: 'The Node version of GIF Sorry Meme online maker'
categories: [tech]
tags: [original]
excerpt: The node version of sorry meme, which is very popular recently, is used to make gif online. The server uses koa + fluent-ffmpeg, and the API uses RESTful specifications.
permalink: /en/:title/
---
> Google Translate from *Chinese version*

> Author: Li Pinghai

Recently, the popular SORRY has produced GIF's Node version. Creative comes from [XUTY's Ruby original](https://github.com/xtyxtyx/sorry). This document also has the original document.

[node-sorry Repo](https://github.com/lipinghai/node-sORRY)

[Demo Address](http://gif.lipinghai.cn/index.html?from=Segmentfault)

If you have any questions, please mention the ISSUE, please star if you like the project, thank you!

![picture](http://imgsrc.baidu.com/forum/pic/item/a98C4BC6A7EFCE1B86B10CCCA351F3DEB58F6585.gif)

## Project Description

* The server uses KOA + FLUENT-FFMPEG to generate subtitles and GIF.
* API uses RESTFUL specification
* Page uses EJS rendering, build script built.js, generate pages and resources in the DIST directory
* The project is configured in config.js (please delete or replace the statistical code for deployment)

## source structure

```
├── package
├── package.lock
├── common                  # Utility classes
├── server                  # Node source code
├── view                    # Page source code
├── template                # GIF templates
├── config.js               # Configuration
├── build.js                # Page build script
├── README.md
└── cache                   # GIF and subtitle cache

```

## Other versions
- [ruby Original](https://github.com/xtyxtyx/sorry), @XUTY writing
- [python version](https://github.com/east196/sorrypy), written by @East196
- [java version](https://github.com/li24361/sorryjava), was written by @li24361
- [C# version](https://github.com/shuangrain/SorryNet), written by @Shuangrain
- [WeChat Mini Program](https://github.com/coxier/iemoji-wechat), written by @CoXier

## API

Make GIF:
```
Post {host}/api/{template_name}/
{
  subtitle: ['Okay', .....]
}

# Return to GIF's Hash
-> 200
{
  status: 200,
  data: 'C2F4069ed207DC38E0F2D9359A2FA6B7'
}
```

Get GIF:
```
Get {host}/api/{template_name}/{gif_hash}
```

The current support Template_name is:
```
- sorry
- wangjingze
```

## Deployment Guild

### Install
```
npm i
```
`@ffmpeg-installer/ffmpeg` may not be installed, pretend more` npm i` a few times

### Build page
```
npm run build
```

### Deployment

Local development `npm run server`

Use recommended `PM2` to manage online, first install the` npm I PM2 -g`, and then `PM2 Start Server/Index.js` can start the project

## Make subtitle template Template.ass

First use AEGISUB to create subtitles for template video, and save it as template.ass (aegisub tutorial.
![picture](https://dn-coding-NET-propuction-P.qbox.me/56a213df-9FF7-9B6C-96b1f0fe2cb6.png))

Then replace the text with a template string < %= Sentences [n] %>
![picture](https://dn-coding-NET-propuction-pp.qbox.me/6b07bc65-4251-aad2-bd7b05af9102.png))