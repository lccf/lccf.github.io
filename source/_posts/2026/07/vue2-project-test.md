---
title: vue2项目迁移选型验证
date: 2026-07-09 22:00
tags:
- vue2
- vite
- farm
- rsbuild
categories: 前端开发
---

最近参与了一些vue2老项目的迭代，启动以及build的速度都不太理想，抽时间看了下不同打包工具的配置及速度对比。

### 结果对比

| 项目            | 第一次(秒) | 第二次(秒) | 第三次(秒) | 平均(秒) |
|----------------|-----------|-----------|-----------|---------|
| vite build     | 3.99      | 3.82      | 3.83      | 3.880   |
| farm build     | 1.56      | 1.26      | 1.24      | 1.353   |
| rsbuild build  | 2.3       | 2.09      | 2.03      | 2.140   |
| vite dev       | 0.169     | 0.166     | 0.169     | 0.168   |
| farm dev       | 0.187     | 0.164     | 0.164     | 0.172   |
| rsbuild dev    | 0.24      | 0.23      | 0.23      | 0.233   |

### vite项目搭建

1.创建项目

```bash
yarn create vite vite-vue2-app -- --template vanilla-ts
yarn add vite@^7 vue@^2 vue-router@^3 element-ui
yarn add -D @vitejs/plugin-vue2
```

2.初始项目文件

```ts
// src/main.ts
import Vue from 'vue';
import App from './App.vue';

const main = () => {
  new Vue({
    render: (h) => h(App)
  }).$mount('#app');
}

main();
```

```vue
<!-- src/App.vue -->
<template>
  <div>{{ msg }}</div>
</template>
<script lang="ts">
import { defineComponent, ref } from 'vue'

export default defineComponent({
  setup() {
    const msg = ref('hello');
    return { msg };
  },
})
</script>
```

```ts
// src/declare.d.ts
declare module '*.vue' {
  import Vue from 'vue';
  export default Vue;
}
```

```ts
// vite.config.ts
import vue from '@vitejs/plugin-vue2'

export default {
  plugins: [vue()]
}
```

3.ai生成验证代码

```markdown
# agent/task.yml
tasks:
  # 读取docs/task.yml文件，找到第一条status不为done的task，执行content中的内容后标记status为done
  # 下一项任务
  # 将任务标记为done
  - name: 添加vue-router相关配置及页面
    status: done
    content: |
      项目添加 vue-router 相关配置
      src/config/router.ts 存放路由配置
      src/App.vue加载router-view
      src/views/home/index.vue 首页
      页面展示 el-table 表格，字段如下：
      - id
      - name
      - desc
      - status 1 启用 -1 禁用
      - created
      - option edit/delete
      表格添加mock数据
  - name: 复制页面添加路由
    status: done
    content: |
      将 home/index.vue 复制 100 份，命名 page1-page100，添加到路由配置中
```

使用ai工具依次执行task生成代码

### rsbuild项目搭建

1.创建项目

```bash
yarn create farm
# 选择vue2
yarn add typescript@~6.0.2 vue-router@^3 element-ui
yarn add -D vite@^7
```

2.编辑项目文件
编辑 index.html 将 script 标签文件修改为 /src/main.ts
删除 src 目录中的文件，将 vite-vue2-app/src 目录中的文件复制到当前项目

### farm项目搭建

1.创建项目

```bash
yarn create rsbuild
# package.json 删除 vue-tsc,@rsbuild/plugin-vue 修改script标签中的命令将 vue-tsc 为 tsc
yarn add vue@^2 vue-router@^3 element-ui
yarn add -D @rsbuild/plugin-vue2
```

2.编辑项目文件

编辑配置文件

```ts
// rsbuild.config.ts
import { defineConfig } from '@rsbuild/core';
import { pluginVue2 } from '@rsbuild/plugin-vue2';

// Docs: https://rsbuild.rs/config/
export default defineConfig({
  html: {
    mountId: 'app'
  },
  source: {
    entry: {
      index: './src/main.ts'
    }
  },
  plugins: [pluginVue2()],
});
```

删除 src 目录中的文件，将 vite-vue2-app/src 目录中的文件复制到当前项目

