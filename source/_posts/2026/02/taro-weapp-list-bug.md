---
title: tarojs开发微信小程序两个bug
date: 2026-02-27 21:07
tags:
- react
- tarojs
- weapp
categories: 前端开发
---

之前维护的项目使用tarojs+react技术栈，记录一下遇到的两个bug及处理方法。

### Bug1：弹窗切换会导致 ScrollView 组件的 scrollTop 被重置

问题描述：当 ScrollView 组件中的内容被滚动以后，点击 switch popup 按钮，切换弹窗之后 ScrollView 滚动的值被重置。
处理方法：给 `{ showPopup && <View>xxx</View> }` 外面再套一层 View，让 Popup 和 ScrollView 在不同的的 层级即可。

示例代码如下：

```typescript
import { Button, Input, ScrollView, View } from "@tarojs/components";
import { useReady } from "@tarojs/taro";
import { useState } from "react";

definePageConfig({
  navigationBarTitleText: "列表弹框",
})

const ListPopupPage = () => {
  const [showList, setShowList] = useState(true);
  const [listData, setListData] = useState<any[]>([]);
  const [showPopup, setShowPopup] = useState(false);

  useReady(() => {
    let result: any[] = [];
    for(let i=0; i<100; i++) {
      result.push({
        title: 'title'+i,
        index: i,
        date: +new Date()
      });
    }
    setListData(result);
  })
  return (<View>
    <Button onClick={()=>setShowPopup(!showPopup)}>switch popup</Button>
    <ScrollView scrollY style={{ height: '800rpx' }}>
      { showList && listData.map((item) => (<View key={ item.index } style={{
        height: '200rpx',
        backgroundColor: '#FF0',
        borderRadius: '20rpx',
        marginTop: '20rpx'
        }}>
        <View>title: { item.title }</View>
        <View>index: { item.index }</View>
      </View>))}
    </ScrollView>
    
    { showPopup && <View style={{
      position: 'fixed',
      left: 0,
      bottom: '100rpx',
      width: '750rpx',
      height:'400rpx',
      backgroundColor: '#FFF'
    }}>
      <Input type="number" confirmType="done" placeholder="请输入文字" />
    </View> }
  </View>);
}

export default ListPopupPage;
```

正确代码如下：

```typescript
import { Button, Input, ScrollView, View } from "@tarojs/components";
import { useReady } from "@tarojs/taro";
import { useState } from "react";

definePageConfig({
  navigationBarTitleText: "列表弹框",
})

const ListPopupPage = () => {
  const [showList, setShowList] = useState(true);
  const [listData, setListData] = useState<any[]>([]);
  const [showPopup, setShowPopup] = useState(false);

  useReady(() => {
    let result: any[] = [];
    for(let i=0; i<100; i++) {
      result.push({
        title: 'title'+i,
        index: i,
        date: +new Date()
      });
    }
    setListData(result);
  })
  return (<View>
    <Button onClick={()=>setShowPopup(!showPopup)}>switch popup</Button>
    <ScrollView scrollY style={{ height: '800rpx' }}>
      { showList && listData.map((item) => (<View key={ item.index } style={{
        height: '200rpx',
        backgroundColor: '#FF0',
        borderRadius: '20rpx',
        marginTop: '20rpx'
        }}>
        <View>title: { item.title }</View>
        <View>index: { item.index }</View>
      </View>))}
    </ScrollView>
    <View>
      { showPopup && <View style={{
        position: 'fixed',
        left: 0,
        bottom: '100rpx',
        width: '750rpx',
        height:'400rpx',
        backgroundColor: '#FFF'
      }}>
        <Input type="number" confirmType="done" placeholder="请输入文字" />
      </View> }
    </View>
  </View>);
}

export default ListPopupPage;
```

### Bug2：页面列表内容过多时 Input 组件输入卡顿

问题描述：当页面中存在过多内容时，页面中的 Input 组件输入会非常卡顿(iPhone手机会更明显)。
处理方法：写一个微信小程序原生组件，在组件中将 Input 的值通过事件触发的方式传到页面。

示例代码如下：

```typescript
import { Button, Input, ScrollView, View } from "@tarojs/components";
import { useReady } from "@tarojs/taro";
import { useState } from "react";

definePageConfig({
  navigationBarTitleText: "列表输入框",
  usingComponents: {
    'custom-input': '../../components/input/input'
  }
})

const ListInputPage = () => {
  const [showList, setShowList] = useState(false);
  const [listData, setListData] = useState([]);

  useReady(() => {
    let result: any[] = [];
    for(let i=0; i<2000; i++) {
      result.push({
        title: 'title'+i,
        index: i,
        date: +new Date()
      });
    }
    setListData(result);
  })
  return (<View>
    <ScrollView scrollY style={{ height: '800rpx' }}>
      { showList && listData.map((item) => (<View key={ item.index } style={{
        height: '200rpx',
        backgroundColor: '#FF0',
        borderRadius: '20rpx',
        marginTop: '20rpx'
        }}>
        <View>title: { item.title }</View>
        <View>index: { item.index }</View>
      </View>))}
    </ScrollView>
    <View style={{
      position: 'fixed',
      left: 0,
      bottom: 0,
      height:'400rpx',
      backgroundColor: '#FFF'
    }}>
      <Input type="number" confirmType="done" placeholder="请输入文字" />
      <custom-input onInput={({ detail }) => {
        console.log('custom-input input:', detail);
      }}/>
      <Button onClick={()=>setShowList(!showList)}>switch list</Button>
    </View>
  </View>);
}

export default ListInputPage;
```

原生组件代码如下：

```xml
<!-- components/input/input.wxml -->
<view style="width: 100%;">
  <view>123123</view>
  <input type="number" confirm-type="done" placeholder="输入组件" bindinput="inputHandler" bindconfirm="confirmHandler" />
</view>
```

```javascript
// components/input/input.js
Component({
  data: {},
  methods: {
    inputHandler: function (e) {
      console.log('input:', e);
      this.triggerEvent('input', e);
    }
  },
})
```

```json
// components/input/input.json
{
  "component": true,
  "usingComponents": {
  }
}
```
