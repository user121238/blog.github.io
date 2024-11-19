---
title: 使用Github Page 和 Hexo 创建blog
tags:
  - Hexo
  - Github Page
  - Fluid
categories:
  - 博客搭建教程
date: 2024-11-19 16:35:37
---

突发奇想想用github page搭建一个个人博客，查了些资料，最终决定使用Hexo。最主要的是这个框架很成熟，而且最最重要的是免费，且支持Markdown和Html，我的Html写的稀烂，用MD刚好。
下面是我搭建这个博客的步骤：

# 安装环境
我使用的Windows搭建，需要安装以下环境：
* Git [[Git官网](https://git-scm.com/)]
* Node.js [[Node.js官网](https://nodejs.org/zh-cn)]


上面的两个工具都直接在官网下载，直接下一步就能完成安装。
> NodeJs安装时要记得勾选  **Add to PATH** ，否则就要自己手动添加环境变量了。

# 安装Hexo
直接把hexo安装成全局的，当然你有自信也可以安装成个人的。
```
npm install -g hexo-cli
```

# 初始化博客项目
在博客源文件存放目录，打开CMD，运行以下命令
```
 hexo init <folder>
 cd <folder>
 npm install
```

命令执行完以后会生成的目录结构如下：
```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

## _config.yml
网站的配置文件，具体参数可以参考官方文档。[hexo官网](https://hexo.io/zh-cn/docs/)

## source
上传的md文档都会在这里面，后面要发布文章主要关注这个文件夹。

**_drafts** 中包含的是草稿，不会显示在网页中。

**_posts** 中是发布的文章，会显示在网页中。


# 运行博客项目

在控制台输入
```
hexo s
```
控制台会输出
```
INFO  Validating config
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/ . Press Ctrl+C to stop.
```
这时候去浏览器访问`http://localhost:4000/` 就可以看到搭建好的博客网站了。

# 上传新文章
上传新文章有两种方式
* 一种是直接将以往的md文件放置在 `source/_posts` 目录中。
* 使用命令

  ```
  hexo new [layout] <title>
  ```
  比如
  ```
  hexo new post 测试文章
  ```
  上面的语句会在`source/_posts/`中生成一个名为 `测试文件.md`的文件，在这个文件中正常写文章就可以发布在网站上了。

  到这里，搭建博客需要的东西就已经准备完成了。

# 更换主题
如果觉得博客自带的主题不好看，可以使用别人制作好的主题，本网站使用了Fluid的主题。

[Fluid主题官网](https://hexo.fluid-dev.com/)

## 安装fluid主题步骤
以下步骤来源于fluid官方文档
1. 安装主题

    进入博客目录，执行命令：
    ```
    npm install --save hexo-theme-fluid
    ```
    然后在博客目录下创建` _config.fluid.yml`，将主题的 [_config.yml](https://github.com/fluid-dev/hexo-theme-fluid/blob/master/_config.yml)内容复制过去。
2. 修改配置文件

    如下修改 Hexo 博客目录中的 _config.yml：
    ```
    theme: fluid  # 指定主题
    language: zh-CN  # 指定语言，会影响主题显示的语言，按需修改
    ```
3. 创建关于页

    首次使用主题的「关于页」需要手动创建：
    ```
    hexo new page about
    ```
    创建成功后修改 `/source/about/index.md`，添加 `layout` 属性。

    修改后的文件示例如下：
    ```
    ---
    title: 标题
    layout: about
    ---

    这里写关于页的正文，支持 Markdown, HTML
    ```
    > layout: about 必须存在，并且不能修改成其他值，否则不会显示头像等样式。

# 部署到github page
部署到github有两种方式，一种是把文件部署到github上，还有一种是一键部署，如果不想将源代码放到github上，可以选择一键部署。
本博客使用文件部署的方式。
1. 创建GitHub仓库
    
   在github上新建仓库，命名方式为  `username.github.io`
   >需要注意的是，仓库名称必须为`username`开头，不然仓库不会识别。

2. 将本地文件push到仓库中
   > `public/`文件夹除外

3. 查看本地的node.js版本
   查询代码如下：
   ```
   node --version
   ```
   记住当前电脑上的node版本
4. 在储存库中建立 .github/workflows/pages.yml，并填入以下内容 (将 20 替换为上个步骤中记下的版本)
    ```
    name: Pages

    on:
    push:
        branches:
        - main # default branch

    jobs:
    build:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v4
            with:
            token: ${{ secrets.GITHUB_TOKEN }}
            # If your repository depends on submodule, please see: https://github.com/actions/checkout
            submodules: recursive
        - name: Use Node.js 20
            uses: actions/setup-node@v4
            with:
            # Examples: 20, 18.19, >=16.20.2, lts/Iron, lts/Hydrogen, *, latest, current, node
            # Ref: https://github.com/actions/setup-node#supported-version-syntax
            node-version: "20"
        - name: Cache NPM dependencies
            uses: actions/cache@v4
            with:
            path: node_modules
            key: ${{ runner.OS }}-npm-cache
            restore-keys: |
                ${{ runner.OS }}-npm-cache
        - name: Install Dependencies
            run: npm install
        - name: Build
            run: npm run build
        - name: Upload Pages artifact
            uses: actions/upload-pages-artifact@v3
            with:
            path: ./public
    deploy:
        needs: build
        permissions:
        pages: write
        id-token: write
        environment:
        name: github-pages
        url: ${{ steps.deployment.outputs.page_url }}
        runs-on: ubuntu-latest
        steps:
        - name: Deploy to GitHub Pages
            id: deployment
            uses: actions/deploy-pages@v4
    ```
    其中`name: Use Node.js 20`这里的20替换成上面查询到的版本号，比如查询到的版本号为`20.13.1`,在此处为`name: Use Node.js 20.13.1`

    



5. 在储存库中前往 `Settings > Pages > Source` 。 将 `source` 更改为 `GitHub Actions`，然后保存。

6. 部署完成，等待系统构建
7. 访问 `username.github.io`查看网页
    >假设你的用户名是：`zhangsan`，那你的仓库名称应该为`zhangsan.github.io`,最终访问的网址也是这个。

# 总结
本教程详细介绍了如何利用Hexo这个静态博客生成器，结合GitHub Pages的免费托管服务，快速搭建一个属于自己的个人博客。通过简单的几步操作，即使是新手也能拥有一个美观、功能强大的博客。