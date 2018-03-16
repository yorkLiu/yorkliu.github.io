---
title: 使用 Hexo 来搭建自己的GitHub Pages 博客
date: 2018-03-16 10:59:06
tags:
  - Hexo
  - GitHub Pages
  - Travis CI
categories:
  - Hexo搭建博客
---

# Hexo
Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](https://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。
> Note: 如果你并不了解 Markdown语法, 请花5分钟阅读 Markdown语法说明: https://www.appinn.com/markdown

# Hexo介绍
[Hexo](https://hexo.io/zh-cn/)是基于NodeJs的静态博客框架，简单、轻量，其生成的静态网页可以托管在Github和Heroku上
* 超快速度
* 支持MarkDown
* 一键部署
* 丰富的插件
* 由于是静态，所以可以部署到 [GitHub Pages](https://pages.github.com/) 上

# 安装
  1. 安装 NodeJs
     Mac OS 下可以使用 [Homebrew](https://github.com/Homebrew/brew) 来安装
     ```bash
     $ brew install node
     ```
  2. 安装hexo
      ```bash
      $ npm install hexo-cli -g
      ```
      **注意** Mac OS 则需要:
      ```bash
      $ sudo npm install hexo-cli -g
      ```
      安装完成之后，执行 `hexo --version`，看到如下信息，恭喜你，你已成功的迈出了艰难的第一步
      ```bash
      hexo-cli: 1.1.0
      node: 8.9.1
      v8: 6.1.534.47
      ...
      ```
# 用Hexo搭建博客
## 创建博客文件夹
  下面以caty.github.io为例
  ```bash
  $ mkdir caty.github.io
  $ hexo init caty.github.io
  $ cd caty.github.io
  $ npm install
  ```
  **注意** 使用`hexo init <folder name>`时, 文件夹必须是一个空的文件夹(没有任何的文件在其中，包括隐藏文件也是不行的)
  > 如果你是用的GitHub 仓库，解决方法也很简单，可以在另一个文件夹中(如： temp)中 用 `hexo init temp`成功之后，再将temp文件中的所以内容Copy到 你的GitHub Repository中即可
    I.E: `$ cp -i temp/* caty.github.io`

## Hexo 命令
- **clean** 删除已生成的数据
- **generate** or **g** 生成HTMl文件
- **server** or **s** 启动server (默认端口为:4000, http://localhost:4000/)
- **deploy** or **d** 发布，如果你配置了GitHub Repository,此命令会将编译好的文件提交至GitHub中
- **new** 创建md文件
  ```bash
  $ hexo new test
  $ echo "This is my first Hexo blog" > test.md
  ```
  此时会在source/_posts目录下生成test.md文件，输入些许内容，然后保存。(当然，你可以直接在 **source/_posts/** 下新建一个.md文件)
  编译一下并启动server, 即可看到第一篇博客页面已经展现在你的眼前了
  ```bash
  $ hexo clean
  $ hexo g
  $ hexo s
  ```
  访问: http://localhost:4000/ 即可看到

## 修改配置
网站的设置大部分都在 **_config.yml** 文件中，详细配置可以查看 [官方文档](https://hexo.io/zh-cn/docs/configuration.html)   
下面列举一些常用的配置：
  - title -> 网站标题
  - subtitle -> 网站副标题
  - description -> 网站描述
  - author -> 您的名字
  - language -> 网站使用的语言(简体中文为 "zh-Hans")
**注意**: 进行配置时，需要在冒号:后加一个英文空格
```
title: My Blog
```
## 标签 Tags
1. 前提条件：
  1. 根目录下的**_config.yml**文件中开启了
    ```
    tag_dir: tags
    ```
  2. 样式(Themes) 目录下的**_config.yml**文件中开启了 (如 thems/next/_config.yml)
    ```
    menu:
      tags: /tags/ || tags
    ```
2. 新建tags页面
  ```
  $ hexo new page tags
  ```
  此时会在source/下生成tags/index.md文件
3. 修改source/tags/index.md
  ```
  title: tags
  date: 2018-03-16 14:49:50
  type: "tags"
  comments: false
  ```
  **注意**: `type`和 `comments`属性**非常**重要
  ```
  type: "tags"
  comments: false
  ```
4. 给文章添加Tags  
  当创建完成 tags之后，就可以在*.md文件中使用了
  ```
  ---
  title: This is Title
  date: 2018-03-16 14:49:50
  tags:
    - Hexo
    - GitHub Pages
    - Travis CI
  ---
  ```

## 类别 Categories
1. 前提条件：
  1. 根目录下的**_config.yml**文件中开启了
    ```
    category_dir: categories
    ```
  2. 样式(Themes) 目录下的**_config.yml**文件中开启了 (如 thems/next/_config.yml)
    ```
    menu:
      categories: /categories/ || th
    ```
2. 新建categories文件
  ```
  $ hexo new page categories
  ```
  此时会在source目录下生成categories/index.md文件
3. 修改source/categories/index.md
  ```
  title: categories
  date: 2018-03-16 14:53:50
  type: "categories"
  comments: false
  ```
  **注意**: `type`和 `comments`属性**非常**重要
  ```
  type: "categories"
  comments: false
  ```
4. 给文章添加 Categories
  当创建完成 categories之后，就可以在*.md文件中使用了
  ```
  ---
  title: This is Title
  date: 2018-03-16 14:53:50
  categories:
    - Hexo搭建博客
  ---
  ```
## 样式 Themes
Hexo 中有很多主题，可以在[官网](https://hexo.io/themes/)查看。 推荐[hexo-theme-next](https://github.com/iissnan/hexo-theme-next)
1. 下载Theme
  ```
  $ git clone https://github.com/iissnan/hexo-theme-next themes/next
  ```
  **注意**如果是用git方式下载，请下载完以后删除`themes/next`目录下的 `.git*`文件
  ```
  rm -rf themes/next/.git*
  ```
2. 使用下载的Theme
  打开根目录下的 **_config.yml**, 配置其中的 **theme**
  ```
  theme: next
  ```
3. theme 配置文件
    每个theme都有自己的 **_config.yml** 文件，路径为: /theme/{theme}/_config.yml，在该文件中可以配置 博客的 Layout方式
    **Scheme** Layout 设置
    推荐使用`scheme: Pisces`

## 添加 搜索 功能
开启博客的 **搜索** 功能
**Install**
```bash
$ npm install hexo-generator-search --save
```
添加以下代码在根目录下的 **_config.yml** 文件中
```
search:
  path: search.xml
  field: post
```
  * path - file path. By default is search.xml . If the file extension is .json, the output format will be JSON. Otherwise XML format file will be exported.
  * field - the search scope you want to search, you can chose:
    * post (Default) - will only covers all the posts of your blog.
    * page - will only covers all the pages of your blog.
    * all - will covers all the posts and pages of your blog

```
$ cd themes/next
vi _config.yml
```
开启`local_search`
```
local_search:
  enable: true
  # if auto, trigger search by changing input
  # if manual, trigger search by pressing enter key or search button
  trigger: auto
  # show top n results per article, show all results by setting to -1
  top_n_per_article: 1
```
## 添加 评价 功能
评价功能推荐使用 [gitment](https://imsun.net/posts/gitment-introduction/)
```
$ cd themes/next
vi _config.yml
```
配置gitment
```
gitment:
  enable: false
  mint: true # RECOMMEND, A mint on Gitment, to support count, language and proxy_gateway
  count: true # Show comments count in post meta area
  lazy: false # Comments lazy loading with a button
  cleanly: false # Hide 'Powered by ...' on footer, and more
  language: # Force language, or auto switch by theme
  github_user: # MUST HAVE, Your Github ID
  github_repo: # MUST HAVE, The repo you use to store Gitment comments
  client_id: # MUST HAVE, Github client id for the Gitment
  client_secret: # EITHER this or proxy_gateway, Github access secret token for the Gitment
  proxy_gateway: # Address of api proxy, See: https://github.com/aimingoo/intersect
  redirect_protocol: # Protocol of redirect_uri with force_redirect_protocol when mint enabled
```
