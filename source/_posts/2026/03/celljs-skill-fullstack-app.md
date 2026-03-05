---
title: 使用celljs skill创建全栈应用
date: 2026-03-05 19:07
tags:
- celljs
- nodejs
- react
categories: 前端开发
---

[celljs](https://github.com/cellbang/cell) 是我很喜欢的一款框架，非常适合用来做全栈(nodejs后端+前端)的项目。介绍一下使用skill来创建一个全栈项目。

### 创建项目

1.创建项目目录

```bash
mkdir celljs-fullstack-app
```

2.安装 celljs-creator skill

从 [https://github.com/lccf/celljs-creator-skill](https://github.com/lccf/celljs-creator-skill) 下载 zip 包，解压后将 celljs-creator 放到项目下的 `.claude/skills` 目录

3.创建任务文档

```yaml
# docs/task.yml
tasks:
  - name: 创建celljs全栈项目
    status: done
    content: |
      调用celljs-creator创建celljs项目
      - celljs版本为3.7.1，所有@celljs下的包版本需保持一致
      - 后端模式为 api
      - 前端框架为 react
      - 手动安装依赖
```

4.打开claude code，确认skills成功加载，输出以下提示语：

```text
读取docs/task.yml取第一条status不为done的task，执行content中的内容后将status标记为done
```

使用的模型为MiniMax-M2.5
根据提示确认文件，完成项目创建，手动安装依赖即可，建议使用yarn

5.目录说明

项目创建完提示如下：

```text
● 已完成celljs项目创建。项目结构如下：                                                                                
                                                                                                                      
  cc-react-fullstack/                                                                                                 
  ├── package.json          # 项目依赖配置                                                                            
  ├── tsconfig.json         # TypeScript配置                                                                          
  ├── cell.yml              # Cell配置                                                                                
  ├── cell-local.yml        # 本地Cell配置                                                                            
  ├── docs/                                                                                                           
  │   └── celljs-refrences.md                                                                                         
  └── src/                                                                                                            
      ├── hooks/
      │   └── webpack.ts    # Webpack钩子配置
      ├── node/            # 后端代码 (api模式)
      │   ├── module.ts
      │   └── controllers/
      └── browser/         # 前端代码 (react)
          ├── module.ts
          ├── root.ts
          ├── app.tsx
          └── service/
```

- 在docs目录创建celljs-refrences.md文件，供ai参考配置数据库链接，添加curd代码，登录认证功能
- src/node 目录存放接口代码，使用 /api/ 路径访问，controllers/home-controller 中有get/post示例代码
- src/browser 目录存放前端代码，service 目录里有访问接口的示例代码

### 链接：
- [celljs-creator-skill](https://github.com/lccf/celljs-creator-skill)
- [celljs](https://github.com/cellbang/cell)
