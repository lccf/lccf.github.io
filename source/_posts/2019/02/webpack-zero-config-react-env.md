---
title: webpack零配置搭建react开发环境
date: 2019-02-26 19:36
tags:
- web
- develop
categories: 前端开发
---

webpack经过多轮的迭代，已经成为前端开发工具链上的霸主，配置也越来越复杂。网上看到了一款插件，可以零配置搭建一个react开发环境，方便单个模块调试。虽然离产品级应用还有距离，却可以节省很多时间和精力，推荐一下。

### 环境搭建

```bash
# 创建目录
mkdir react-app
cd react-app
# 创建配置文件
echo '{}' > package.json
# 安装webpack及依赖
npm i -D webpack webpack-dev-server webpack-cli html-webpack-plugin common-config-webpack-plugin
# 安装react
npm i -S react react-dom
```

### react编码

编辑src/index.jsx

```javascript
# src/index.jsx
import React, { Component } from 'react';
import ReactDOM from 'react-dom';

class App extends Component {
  constructor(props) {
    super(props);
  }
  render() {
    let { text } = this.props;
    return (<div>{text}</div>);
  }
}

const renderEl = (elementId = 'app') => {
  let el = document.getElementById(elementId);
  if (!el) {
    el = document.createElement('div');
    el.setAttribute('id', elementId);
    document.body.appendChild(el);
  }
  ReactDOM.render(<App text="React Demo" />, el);
  el = null;
}

renderEl();
```

### 启动项目

```bash
npx webpack-dev-server --plugin common-config-webpack-plugin --plugin html-webpack-plugin
```

访问 http://localhost:8080 即可看到效果。

### 快速启动

编辑package.json，将启动命令配置到scripts中，即可使用npm start启动项目。完整package.json如下：

```json
{
  "scripts": {
    "start": "webpack-dev-server --plugin common-config-webpack-plugin --plugin html-webpack-plugin"
  },
  "devDependencies": {
    "common-config-webpack-plugin": "^1.3.1",
    "html-webpack-plugin": "^3.2.0",
    "webpack": "^4.29.5",
    "webpack-cli": "^3.2.3",
    "webpack-dev-server": "^3.2.1"
  },
  "dependencies": {
    "react": "^16.8.3",
    "react-dom": "^16.8.3"
  }
}
```

启动命令

```bash
npm start
```

### 链接
webpack-config-plugins https://github.com/namics/webpack-config-plugins
react https://reactjs.org/
