---
title: 说说redux-actions库
date: 02/26/2019 23:21
tags:
- web
- develop
categories: 前端开发
---

接上一篇 [webpack零配置搭建react开发环境](./webpack-zero-config-react-env) 说一说redux-actions这个库。

### 安装redux及相关库

```bash
npm i -S redux redux-actions redux-promise
```

- redux-action 示例action的相关工具主法
- redux-promise 是一个异步库搭配redux-actions使用

### redux counter示例

src/index.jsx
```javascript
import React from 'react';
import ReactDOM from 'react-dom';

import App from './app';

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

src/app.jsx
```javascript
import React, { Component } from 'react';
import { createStore } from 'redux';

import { actionIncrement, actionDecrement, counterReducer } from './plainState';

let initState = { counter: 0 };

export default class App extends Component {
  constructor(props) {
    super(props);
    let store = createStore(counterReducer, initState);
    let { getState, dispatch } = store;
    this.state = getState();
    this.dispatch = dispatch;

    store.subscribe(() => this.setState(getState()));
  }
  render() {
    let { text } = this.props;
    let { counter } = this.state;
    return (<div>
      <p>text:{ text }</p>
      <p>counter:{ counter }</p>
      <div>
        <button type="button" onClick={ ()=>this.dispatch(actionIncrement()) }>Increment</button>
        <button type="button" onClick={ ()=>this.dispatch(actionDecrement()) }>Decrement</button>
      </div>
    </div>);
  }
}
```

src/painState.js

```javascript
export const ACTION_INCREMENT = 'ACTION_INCREMENT';
export const actionIncrement = (amount = 1) => ({type: ACTION_INCREMENT, payload: amount});

export const ACTION_DECREMENT = 'ACTION_DECREMENT';
export const actionDecrement = (amount = -1) => ({type: ACTION_DECREMENT, payload: amount});

export const counterReducer = (state, { type, payload }) => {
    let { counter } = state;
    if (type == 'ACTION_INCREMENT') {
        counter += payload;
    }
    else if (type == 'ACTION_DECREMENT') {
        counter -= payload;
    }
    else {
        counter = 0;
    }
    return { ...state, counter };
}
```
