---
title: umi3过滤路由优化项目启动
date: 2026-01-28 18:30
tags:
- web
- develop
- umijs
- umi3
categories: 前端开发
---

老项目使用umi3+antd pro架构、约定式路由，项目长时间迭代页面越来越多，每次启动及修改后的加载时间都在增加。综合了几个方案最终选择了改动最小的方式，通过umijs插件动态修改路由，开发启动项目时只加载指定路由的页面，大大提高了开发效率。插件代码及配置如下：

### 插件代码

```typescript
// src/plugins/routeFilter.ts

import { IApi } from 'umi';

export default function (api: IApi) {
  api.describe({
    key: 'routeFilter',
    config: {
      schema(joi) {
        return joi.object({
          exclude: joi.array().items(joi.string()),
          include: joi.array().items(joi.string()),
          modify: joi.function(),
          strict: joi.boolean().default(false),
        });
      },
    },
  });

  api.modifyRoutes((originalRoutes: any[]) => {
    const { exclude, include, modify, strict = false } = api.config.routeFilter || {};

    // 深度优先遍历路由树
    const filterRoutes = (routes: any[]): any[] => {
      return routes
        .map((route) => ({ ...route })) // 浅拷贝路由对象
        .filter((route) => {
          // 1. 处理排除规则
          if (exclude && exclude.includes(route.path)) {
            return false;
          }

          // 2. 处理包含规则
          if (include) {
            const shouldInclude = include.includes(route.path);
            if (strict) {
              return shouldInclude;
            }
            // 非严格模式下，如果路由有子路由，继续检查子路由
            if (!shouldInclude && route.routes) {
              return true;
            }
            return shouldInclude;
          }

          return true;
        })
        .map((route) => {
          // 递归处理子路由
          if (route.routes) {
            const filteredChildren = filterRoutes(route.routes);
            // 如果过滤后子路由为空，根据配置决定是否保留父路由
            if (filteredChildren.length === 0 && strict) {
              return null;
            }
            return { ...route, routes: filteredChildren };
          }
          return route;
        })
        .filter(Boolean); // 过滤掉被标记为null的路由
    };

    let filteredRoutes = filterRoutes(originalRoutes);

    // 3. 应用自定义修改函数
    if (modify && typeof modify === 'function') {
      filteredRoutes = modify(filteredRoutes);
    }

    return filteredRoutes;
  });
}
```

- 将代码保存至`src/plugins/routeFilter.ts`文件
- 插件代码使用deepseek生成
- 仅适用于umi3，umi4的routes结构从array改为key/value方式，代码需要相应修改

### 项目配置

1.创建`.umirc.quick.ts`

```typescript
// .umirc.quick.ts
export default {
  routeFilter: {
    // 排除特定路径的路由
    // exclude: ['/admin', '/secret'],

    // 或者只包含特定路径的路由
    // include: [
    //   '/home'
    //  ],

    strict: false,

    // 自定义修改函数
    modify: (routes: any[]) => {
      // 是否为嵌套路由，未使用全局路由时，此处应该置为false
      let nestedRoute = true;
      let exactList: string[] = [
        '/'
      ];
      let includeList: string[] = [
        '/home',
        // '/User',
        // '/Controlcabin',
        // '/Management',
        // '/noaccess',
        //  '/contract',
      ];
      let excludeList: string[] = [
        // '/contract'
      ];
      let includeHandler = (item: any) => {
        if (item.path) {
          let hasInclude = includeList.filter((key) => item.path.indexOf(key) > -1);
          if (hasInclude?.length > 0 || exactList.includes(item.path)) {
            return true;
          }
        }
        return false;
      };
      let excludeHandler = (item: any) => {
        if (item.path) {
          let hasExclude = excludeList.filter((key) => item.path.indexOf(key) > -1);
          if (hasExclude?.length > 0) {
            return false;
          }
        }
        return true;
      };

      if (nestedRoute) {
        // 可以在这里对路由进行更复杂的修改
        return routes.map((route) => {
          if (route.routes?.length) {
            try {
              if (includeList.length) {
                route.routes = route.routes.filter(includeHandler);
              }
              if (excludeList.length) {
                route.routes = route.routes.filter(excludeHandler);
              }
            } catch (err) {
              console.error(err);
            }
          }
          return route;
        });
      }
      else {
        try {
          if (includeList.length) {
            routes = routes.filter(includeHandler);
          }
          if (excludeList.length) {
            routes = routes.filter(excludeHandler);
          }
        } catch (err) {
          console.error(err);
        }
        return routes;
      }
    },
  },
  plugins: ['./src/plugins/routeFilter'],
};
```

- 使用全局`layout`页面时`modify`方法中的`nestedRoute`设置为`true`
- `includeList`配置允许的路径，`/home`允许所有以`/home`开头的路径(区分大小写)
- `excludeList`配置排除的路径，规则同`includeList` 

2.修改`package.json`文件
在`scripts`中添加启动方式`"start-quick": "cross-env UMI_ENV=quick umi dev"`

3.使用`yarn start-quick`启动项目
仅在使用`yarn start-quick`命令时配置生效，不影响其它命令

### 链接
- modifyRoute [https://v3.umijs.org/zh-CN/plugins/api#modifyroutes](https://v3.umijs.org/zh-CN/plugins/api#modifyroutes)
