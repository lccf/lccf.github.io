---
title: 使用vibe coding开发celljs登录鉴权和单表curd
date: 2026-04-02 19:00
tags:
- celljs
- nodejs
- react
categories: 前端开发
---

在上次 [使用celljs skill创建全栈应用](../03/celljs-skill-fullstack-app.html) 的基础项目上做一个简单的后台内容管理功能，实现登录鉴权和单表curd。目前ai编码工具的推理能力很强，在给定参照的情况下，可以按照给定的规则和编码方式去输出。实际的使用中还需要人工监督，有针对性的反馈报错进行处理。

### 添加用户登录功能

1.编辑 docs/task.yml 添加提示词

```yaml
tasks:
  # 创建celljs全栈项目
  - name: 添加用户认证
    status: none
    content: |
      安装依赖时@celljs开头的包版本需保持一致
      参考 `docs/celljs-refrences.md` 给项目添加数据库支持
      - 数据库使用 sqlite
      - 数据库路径 db/main.db
      参考 `docs/celljs-refrences.md` 添加用户认证
      - jwt secret 为 123456
      - 用户实体字段如下：
        - 主键(id)
        - 用户名(username)
        - 密码(password)
        - 邮箱(email)
        - 备注(desc)
        - 创建时间(created_at)
        - 更新时间(updated_at)
      创建 user-controller 控制器，添加 setup 方法，创建默认用户 用户名 admin 密码 123456 邮箱 admin@user.com
  - name: 添加前端依赖和页面
    status: none
    content: |
      添加 react-router、antd、antd pro、antd icons依赖
      前端代码目录为 src/browser
      - 参考 src/browser/service/home-service.ts 实现登录请求，请求地址为 `/api/login`，接口由npm提供
      - 登录接口返回的数据结构参考 src/common/response-data 中的定义，token 在 data.token 
      - 添加 login 页面，输入用户名和密码点击登录， 登录成功后将 token 存储到 localstorage
      - 添加 dashboard 页面，需要用户登录后才能访问
      - 修改 HttpClient 添加 Put、Delete方法
      - 修改 HttpClient 如果本地存在 token，则在请求 header 中带入 token
  - name: 设计一下dashboard和login页面
    status: none
    content: 这是一个内容管理系统，设计一下dashboard和login页面，使用浅色主题
```

3.初始化用户

在 claude code 中输入 next task 添加用户及登录相关代码

```bash
# 安装依赖
yarn install
# 启动项目并难
yarn start
# 创建默认用户
curl -i http://localhost:3000/api/users/setup
# Ctrl-c 停止项目
```

4.添加登录界面

```bash
# 初始化界面skill
uipro init --ai claude
```

在 claude code 中输入 next task 添加登录界面

```bash
# 安装依赖
yarn install
# 启动项目并验证
yarn start
# Ctrl-c 停止项目
```

在 claude code 中输入 next task 美化登录界面

```bash
# 启动项目并验证
yarn start
# Ctrl-c 停止项目
```

### 添加内容管理页面

1.编辑 docs/task.yml 文件，添加内容管理的系统

```yaml
tasks:
  # 设计一下dashboard和login页面
  - name: 创建内容表和控制器
    status: none
    content: |
      参考 `docs/celljs-refrehces.md` 创建 content 实体，字段如下：
        - 主键(id)
        - 标题(title)
        - 内容(content)，长文本
        - 备注(desc)
        - 可见(visible)，布尔值
        - 创建时间(created_at)
        - 更新时间(updated_at)
      创建 content-controller 实现 content 表的 curd 接口，接口返回格式参考 src/common/response-data
      - create、update、delete 接口需要校验登录
  - name: 添加内容前端页面
    status: none
    content: |
      添加内容前端页面
      - 设计 dashboard 页面，标题栏下方分为左右两块，左侧展示菜单，右侧展示内容
      - 在左侧菜单添加 内容管理 链接，点击右方展示内容管理页面
      - 内容管理页面用表格展示 content 列表，操作栏 展示 编辑、删除
      - 点击编辑使用弹窗编辑当前 content ，点击确定保存
      - 点击删除，提示确认弹窗，点击确认删除当前 content
      - 在表格上方添加 新建内容 按钮，点击后在弹窗中可以新建内容
```

2.应用修改

在 claude code 中 next task 依次完成 task

```bash
# 启动项目并验证
yarn start
# Ctrl-c 停止项目
```

### 链接：
- [celljs](https://github.com/cellbang/cell)
- [celljs认证](https://cell.naily.cc/components/security/example)
- [celljs typeorm](https://cell.naily.cc/components/database/typeorm)
