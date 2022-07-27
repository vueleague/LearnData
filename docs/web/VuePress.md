---
article: false
title: VuePress
order: 1
---

[VuePress](https://v2.vuepress.vuejs.org/zh/guide/getting-started.html) 是一个以 Markdown 为中心的静态网站生成器。你可以使用 Markdown 来书写内容（如文档、博客等），然后 VuePress 会帮助你生成一个静态网站来展示它们。

如果你已经有了 docsify/Hexo 等 Markdown 架构网站，简单就能转为 VuePress。不过，VuePress 网站需要依赖包环境，生成的静态文件在本地运行会缺少组件，只推荐在服务器或其他云服务上运行。

主题使用的是 [vuepress-theme-hope](https://vuepress-theme-hope.github.io/v2/zh/guide/)，其他主题和插件查看 [Awesome VuePress V2](https://github.com/vuepress/awesome-vuepress/blob/main/v2.md)。

## 插件

- [docsearch](https://v2.vuepress.vuejs.org/zh/reference/plugin/docsearch.html)：将 Algolia DocSearch 集成到 VuePress 中，为你的文档网站提供搜索功能。开源技术向博客可以申请官方支持，商业化内容需要使用自己的爬虫。
- [Iconfont 精选图标](https://vuepress-theme-hope.github.io/v2/zh/guide/interface/icon.html#iconfont-%E7%B2%BE%E9%80%89%E5%9B%BE%E6%A0%87)
- [看板娘](https://www.npmjs.com/package/vuepress-plugin-helper-live2d)

## 初始配置

1. 环境配置：安装 npm、yarn、pnpm，方法查看 [Linux 环境部署教程](../deploy/VPS.html#环境部署)。
2. 新建文件夹，然后在该路径下运行命令 `pnpm create vuepress-theme-hope@next docs`。vuepress-theme-hope 主题的样例文件会存储在该路径下。
3. 执行命令 `pnpm docs:dev` 启动样例网站。
4. `docs\.vuepress` 路径下的 config.ts，navbar.ts，sidebar.ts，theme.ts 可以修改页面属性，设置方法参考 [官方案例](https://github.com/vuepress-theme-hope/vuepress-theme-hope/tree/main/docs/theme/src/.vuepress)。
   - config.ts：环境依赖包
   - sidebar.ts：侧边栏，集合所有文档的目录
   - navbar.ts：导航栏，放最常用的文档链接
   - theme.ts：对主题和插件进行设置

## 固定静态文件名

VuePress v2 默认使用 Vite 打包，静态文件名会根据 hash 自动生成。这导致打包总会替换网站大部分的静态文件，对服务器的自动部署。按 [vue.config.js](https://cli.vuejs.org/config/#vue-config-js) 的配置添加 `filenameHashing: false`，但并未停止生成 hashname。

因此，我把打包工具更换为 [Webpack](https://v2.vuepress.vuejs.org/zh/guide/bundler.html)，并用 chainWebpack 设置静态名生成规则。

1. 修改 config.ts 的导入设置，将 `import { defineUserConfig } from "vuepress";` 替换为 `import { defineUserConfig } from "@vuepress/cli";`。

2. Webpack 环境依赖包安装，并运行服务。

   ```bash
   #组合命令，打包使用 Webpack
   pnpm add vuepress@next vuepress-theme-hope@next && pnpm remove vuepress && pnpm add vuepress-webpack@next sass-loader && pnpm i && pnpm up

   #运行在本地服务器
   yarn docs:dev
   ```

   组合命令也能**解决报错**，升级相关依赖包。相关命令的分步解释见下方。

   ```bash
   #确保你正在使用最新的 vuepress 和 vuepress-theme-hope 版本
   pnpm add vuepress@next vuepress-theme-hope@next

   #更换打包工具，Webpack 需手动下载 sass-loader
   pnpm remove vuepress
   pnpm add -D vuepress-webpack@next sass-loader

   #常用插件：google-analytics，docsearch
   pnpm add @vuepress/plugin-google-analytics@next @vuepress/plugin-docsearch@next

   #升级当前目录的依赖以确保你的项目只包含单个版本的相关包
   pnpm i && pnpm up
   ```

3. 固定 js 静态文件名：打开 config.ts，使用 [webpack-chain](https://github.com/Yatoo2018/webpack-chain/tree/zh-cmn-Hans) 修改 webpack 输出文件名规则，停止对数量最多的 chunk 文件 hashname。^[[chainWebpack 长用配置方式](https://blog.csdn.net/song854601134/article/details/121340077)]

   ```ts
   export default defineUserConfig({
     bundler: webpackBundler({
       chainWebpack(config) {
         // do not use chunk hash in js
         //参照案例：https://github.com/vuepress/vuepress-plugin-named-chunks/blob/b9fb5a1d3475530b1d74b6616f92a6e3bf14a7ed/__tests__/docs/.vuepress/config.js
         config.output.chunkFilename("assets/chunks/[name].js");
       },
     }),
   });
   ```

## 关闭 prefetch

preload 是一种声明式的资源获取请求方式，用于提前加载一些需要的依赖，并且不会影响页面的 onload 事件。prefetch 是一种利用浏览器的空闲时间加载页面将来可能用到的资源的一种机制；通常可以用于加载非首页的其他页面所需要的资源，以便加快后续页面的首屏速度。preload 主要用于预加载当前页面需要的资源；而 prefetch 主要用于加载将来页面可能需要的资源。

VuePress [Build 配置项](https://vuepress.github.io/zh/reference/config.html#build-%E9%85%8D%E7%BD%AE%E9%A1%B9) 默认开启了 preload 和 prefetch。但是，开启了 prefetch，所有其它页面所需的文件都会被预拉取。页面较多或服务器宽带后付费的话，建议关闭 prefetch。

`docs\.vuepress` 路径下的 config.ts 配置中插入 `shouldPrefetch: false,`，即可关闭 prefetch。

## 自定义主题

- [ ] [waline](https://vuepress-theme-hope.github.io/v2/zh/guide/feature/comment.html#waline) 评论插件
- [ ] Algolia DocSearch 申请中，等结果通知
- [x] ~~google analytics 没反应，实际已经包含在 js 中了~~
- [x] ~~不用自动开启一堆的网站，关闭 prefetch~~
- [x] ~~生成文件名固定化，chainWebpack~~
- [x] ~~网页更新时，有时会打不开链接，需要使用缓存。~~
- [x] ~~VuePress 博客页面：frontmatter 中添加 order 参数让最新的文章往上排，无法按文件名倒序排列~~
- [x] 全局路径需要给子目录添加 README.md，没那么多内容填，暂时放弃。
- [x] 独立设置页面标题。未成功，所有页面都会加入默认标题。
- [x] 侧边栏显示客服图片。icon 位置直接放链接也没用。
- [x] 指定页面子标题不被目录页识别。但页面中取消 toc 的话，网页位置会出现偏移。
- [x] [修改导航栏 brand 链接](https://vuepress-theme-hope.github.io/v2/zh/cookbook/advanced/replace.html#%E6%8F%92%E6%A7%BD%E5%88%A9%E7%94%A8)，需用本地文件替代 [主题默认设置](https://github.com/vuepress-theme-hope/vuepress-theme-hope/blob/main/packages/theme/src/client/module/navbar/components/NavbarBrand.ts)。设置的 ts 未生效，暂时放弃。
- [x] ~~子域名中部署 blog 和 note，分别使用不同路径。这方案可以与 WordPress 共存，但未了避免后续出错，还是取消了。~~
- [x] ~~Giscus 评论区设置~~
- [x] ~~导航栏添加 repo 位置~~
- [x] ~~页面统计，插件只支持 Google、百度，后用图片标签方式植入统计。备用方法：将统计代码直接放在侧边栏。~~
- [x] ~~定制页面标签：config.ts 中添加全局 [head 标签](https://github.com/vuepress-theme-hope/vuepress-theme-hope/blob/main/docs/theme/src/.vuepress/config.ts)，或在页面中添加 [独立 head 标签](https://vuepress-theme-hope.github.io/v2/seo/zh/guide.html#%E7%9B%B4%E6%8E%A5%E6%B7%BB%E5%8A%A0-head-%E6%A0%87%E7%AD%BE)，支持图片统计代码。~~
- [x] ~~将 docs 里的 README.md 转移到主目录中，保持 github 项目页的同步。~~
- [x] ~~打开页面链接，侧边栏焦点能不能也移动过去。侧边栏标题需要能在首屏出现，才能激活焦点。~~
- [x] ~~默认主题颜色为白天，虽然不能切换，但发稿用白色就性。~~
- [x] ~~设置导航栏自动隐藏~~
- [x] ~~隐藏编辑时间和贡献者~~
- [x] ~~用 md 控制图片是，图片不能显示。这可能是因为主题默认的 lazy 加载，改用七牛云的图片属性控制尺寸。~~