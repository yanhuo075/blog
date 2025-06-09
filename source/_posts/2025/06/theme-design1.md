---
title: Hexo-Stellar主题装修日记(一)
categories: [装修日记]
tags: [主题装修]
---

## 前言

记录一下自己从默认主题样式到现在本网站的样子中间都修改了哪些地方，也作为完善博客期间的备份。

## 博客引用资源

### 本地资源文件夹

将头像、图标、css、js等文件放在source下

### 静态资源本地化

有两次发现通过jsdelivr引入的静态资源会失效，为了网站的稳定性，决定将所有**通过jsdelivr引入的静态资源**保存到本地，防止失效后网站无法访问。

### 引用落霞孤鹜字体

```
tale/_config.ymlinject:
  head:
    - <link rel="stylesheet" href="https://npm.elemecdn.com/lxgw-wenkai-screen-webfont/style.css" media="print" onload="this.media='all'"> #字体

Copy
tale/_config.stellar.ymlstyle:
  font-family:
    body: '"LXGW WenKai Screen", sans-serif'

Copy
```

### 引用鸿蒙字体

```
tale/_config.ymlinject:
  head:
    - <link rel="preconnect" href="https://s1.hdslb.com/" /> 
    - <link rel="stylesheet" href="//s1.hdslb.com/bfs/static/jinkela/long/font/regular.css" media="all" onload="this.media='all'" /> #鸿蒙正常字体
    - <link rel="stylesheet" href="//s1.hdslb.com/bfs/static/jinkela/long/font/medium.css" media="all" onload="this.media='all'" /> #鸿蒙加粗字体

Copy
tale/_config.stellar.ymlstyle:
  font-family:
    body: 'HarmonyOS_Regular' # 鸿蒙正常字体
    #body: 'HarmonyOS_Regular' # 鸿蒙加粗字体

Copy
```

## 博客基本配置

### 基本信息

```
tale/_config.yml# Site
title: SFZhang #网站名称
avatar: /customize/images/sfzhang.jpg #博客头像
favicon: /customize/images/sfzhang.jpg #网站图标
subtitle: "SFZhang's blog | blog.sfzhang.cn" #鼠标移入翻转效果
description: '一个积极生活的人'
keywords:
author: SFZhang #博客作者
language: zh-CN #博客语言：en、zh-CN
timezone: ''

Copy
```

### 显示导航

```
tale/_config.stellar.ymlsidebar:
  menu:
    post: '[btn.blog](/)'
    wiki: '[btn.wiki](/)'
    friends: '[友链](/)'
    about: '[关于](/)'

Copy
```

### 图片放大（fancybox）

其中selector设置为需要放大图片的HTML选择器：

```
tale/_config.ymltag_plugins:
  # {% image %}
  image:
    fancybox: true # true, false
    parse_markdown: true # 把 markdown 格式的图片解析成图片标签

######## JS Plugins ########
plugins:
  # https://fancyapps.com/docs/ui/fancybox/
  # available for {% image xxx %}
  fancybox:
    enable: true
    #js: /customize/js/fancybox.umd.js
    css: /customize/css/fancybox.css
    js: https://fastly.jsdelivr.net/npm/@fancyapps/ui@4.0/dist/fancybox.umd.js
    #css: https://fastly.jsdelivr.net/npm/@fancyapps/ui@4.0/dist/fancybox.css
    # 可以处理评论区的图片（不支持 iframe 类评论系统）例如：
    # 使用twikoo评论可以写: .tk-content img:not([class*="emo"])
    # 使用waline评论可以写: #waline_container .vcontent img
    selector: .swiper-slide img, .md-text.content p>img, .md-text.content li img , .wl-content img, .image-bg img # 多个选择器用英文逗号隔开
    #selector: .swiper-slide img # 多个选择器用英文逗号隔开

Copy
```

## 侧边栏

### 侧边栏底部按钮

```
tale/_config.stellar.ymlfooter:
  social:
    QQ:
      icon: '<img src="/customize/svg/contact.svg"/>'
      url: https://wpa.qq.com/msgrd?v=3&uin=1119716380&site=qq&menu=yes&jumpflag=1
    github:
      icon: '<img src="/customize/svg/github.svg"/>'
      url: https://github.com/z23654262
    #iconfont:
    #  icon: '<img src="/customize/svg/iconfont.svg"/>'
    #  url: https://www.iconfont.cn/
    unsplash:
      icon: '<img src="/customize/svg/unsplash.svg"/>'
      url: https://unsplash.com/@z23654262
    train:
      icon: '<img src="/customize/svg/train.svg"/>'
      url: https://unsplash.com/@z23654262
    Moon:
      icon: '<img id="ThemeM" src="/customize/svg/moon.svg"/>'
      url: javaScript:void('永夜');
    Sun:
      icon: '<img id="ThemeL" src="/customize/svg/sun.svg"/>'
      url: javaScript:void('永昼');
    #AI:
    # icon: '<img id="ThemeAI" src="/customize/svg/auto.svg"/>'
    # url: javaScript:void('跟随系统');

Copy
```

### 小组件

```
tale/source/_data/widget,ymlwelcome:
  layout: markdown
  title: '欢迎来到SFZhang的知识库'
  content: |
    不以物喜，不以己悲
    <script src="https://v1.hitokoto.cn/?c=i&encode=js&select=%23hitokoto" defer=""></script>
    <p>一诗：<span id="hitokoto"></span></p>

yiyan:
  layout: markdown
  title: '一诗'
  content: |
    <script src="https://v1.hitokoto.cn/?c=i&encode=js&select=%23hitokoto" defer=""></script>
    <p><span id="hitokoto"></span></p>

timeline:
  layout: timeline
  title: 生活碎片
  api: https://api.github.com/repos/z23654262/blog-life/issues?per_page=10 # 若你想限制数量，在api链接后面加上?per_page=1指限制为1条
  user: # 是否过滤只显示某个人发布的内容，如果要筛选多人，用英文逗号隔开
  hide: # title,footer # 隐藏标题或底部 # 此功能需要 Stellar v1.13.0

ghuser:
  layout: ghuser
  username: z23654262 # your github login username
  avatar: true # show avatar or not
  menu: true # show menu or not

#搜索
search_blog:
  layout: search
  filter: /blog/ # or /posts/ ...
  placeholder: 文章搜索

search_all_docs:
  layout: search
  filter: /wiki/
  placeholder: 文档系统搜索

search_docs:
  layout: search
  filter: auto
  placeholder: 文档内搜索

Copy
```

## 生活碎片

### 添加页面



### 添加首页按钮

```
tale/_config.stellar.ymlpost-index: # 近期发布 分类 标签 归档 and ...
  '生活碎片': /life

Copy
```

### 导航高亮问题

```
tale/node_modules/hexo-theme-stellar/layout/_partial/main/navbar/list_post.ejsif (full_url_for(page.path) == full_url_for(obj[key]))

Copy
```

修改为

```
tale/node_modules/hexo-theme-stellar/layout/_partial/main/navbar/list_post.ejsif (full_url_for(page.path) == full_url_for(obj[key]) + '/index.html')

Copy
```

### 新建github仓库并添加一条issue

![img](https://bu.dusays.com/2023/10/10/6525053ada517.png)

### 编辑生活碎片页面

```
tale/source/life/index.md---
title: 
menu_id: post #侧边栏首页高亮
breadcrumb: false # 隐藏面包屑导航
post_list: true # 显示首页导航
date: 2023-10-10 10:34:34
---

{% note color:orange 分享自己的生活碎片！ %}

{% timeline api:https://api.github.com/repos/z23654262/blog-life/issues?direction=asc&per_page=30 %}{% endtimeline %}

Copy
```

### 预览生活碎片页面

![img](https://bu.dusays.com/2023/10/10/6525053c629b0.png)

## 添加评论系统giscus

### 新增github仓库

新增blog-comments仓库并开启Discussions功能并勾选**blog-comments/Settings/Features/Discussions**即可。

### 修改配置

```
tale/_config.stellar.ymlcomments:
  service: giscus
  # giscus
  # https://giscus.app/zh-CN
  giscus:
    data-repo: xxx/xxx # [在此输入仓库]
    data-repo-id: # [在此输入仓库 ID]
    data-category: # [在此输入分类名]
    data-category-id:
    data-mapping: pathname
    data-strict: 0
    data-reactions-enabled: 1
    data-emit-metadata: 0
    data-input-position: top # top, bottom
    data-theme: preferred_color_scheme
    data-lang: zh-CN
    data-loading: lazy
    crossorigin: anonymous

Copy
```

### 测试giscus评论系统

博客内评论：

![img](https://bu.dusays.com/2023/10/10/652506ecef464.png)

github仓库中Discussions：

![img](https://bu.dusays.com/2023/10/10/65250743df7b0.png)

## 添加waline评论系统

[waline官方教程](https://waline.js.org/guide/get-started/)

waline程序托管于vercel，数据存储使用learncloud国际版，域名使用waline.sfzhang.top二级域名

### learncloud国际版设置

1. [登录](https://console.leancloud.app/login) 或 [注册](https://console.leancloud.app/register) `LeanCloud 国际版` 并进入 [控制台](https://console.leancloud.app/apps)

2. 点击左上角`创建应用`并起一个你喜欢的名字 (请选择免费的开发版)

   ![img](https://bu.dusays.com/2023/10/25/653934aa64f92.png)

3. 进入应用，选择左下角的 `设置` > `应用 Key`。你可以看到你的 `APP ID`,`APP Key` 和 `Master Key`。请记录它们，以便后续使用。

   ![img](https://bu.dusays.com/2023/10/25/653934abc41c2.png)

### vercel设置

1. 点击[Vercel](https://vercel.com/new/clone?repository-url=https://github.com/walinejs/waline/tree/main/example)，跳转至 Vercel 进行 Server 端部署。（如果登录无法访问github，则使用邮箱登录）

2. 跳转后会自动机遇waline仓库进行初始化，只需要填写Vercel仓库名称即可。

   ![img](https://bu.dusays.com/2023/10/25/653939ea76abe.png)

3. 点击顶部的 `Settings` - `Environment Variables` 进入环境变量配置页，并配置三个环境变量 `LEAN_ID`, `LEAN_KEY` 和 `LEAN_MASTER_KEY` 。它们的值分别对应上一步在 LeanCloud 中获得的 `APP ID`, `APP KEY`, `Master Key`。

   ![img](https://bu.dusays.com/2023/10/25/653939ec3590b.png)

4. 环境变量配置完成之后点击顶部的 `Deployments` 点击顶部最新的一次部署右侧的 `Redeploy` 按钮进行重新部署。该步骤是为了让刚才设置的环境变量生效。

   ![img](https://bu.dusays.com/2023/10/25/653939edb10db.png)

5. 此时会跳转到 `Overview` 界面开始部署，等待片刻后 `STATUS` 会变成 `Ready`。此时请点击 `Visit` ，即可跳转到部署好的网站地址，此地址即为你的服务端地址。

### Vercel绑定域名

1. 点击顶部的 `Settings` - `Domains` 进入域名配置页，并输入需要绑定的域名

   ![img](https://bu.dusays.com/2023/10/25/653939ef268d9.png)

2. 在阿里云处配置域名解析

   ![img](https://bu.dusays.com/2023/10/25/653939f04aa4b.png)

### 博客配置文件

```
tale/_config.staller.yml######## Comments ########
comments:
  service: waline # beaudar, utterances, giscus, twikoo, waline, artalk
  
  # Waline
  # https://waline.js.org/
  waline:
    js: https://unpkg.com/@waline/client@2.14.1/dist/waline.js
    css: https://unpkg.com/@waline/client@2.14.1/dist/waline.css
    # Waline server address url, you should set this to your own link
    serverURL: https://domain
    # If false, comment count will only be displayed in post page, not in home page
    commentCount: true
    # Pageviews count, Note: You should not enable both `waline.pageview` and `leancloud_visitors`.
    pageview: false
    # Custom emoji
    # emoji:
    #   - https://unpkg.com/@waline/emojis@1.1.0/weibo
    #   - https://unpkg.com/@waline/emojis@1.1.0/alus
    #   - https://unpkg.com/@waline/emojis@1.1.0/bilibili
    #   - https://unpkg.com/@waline/emojis@1.1.0/qq
    #   - https://unpkg.com/@waline/emojis@1.1.0/tieba
    #   - https://unpkg.com/@waline/emojis@1.1.0/tw-emoji
    #   - https://unpkg.com/@waline/emojis@1.1.0/bmoji
    # 设置自己的图床服务，替换默认的 Base 64 编码嵌入（有体积大小限制），在评论中上传图片更加方便
    # imageUploader:
      # 适配了兰空图床V1、V2版本
      # 以兰空图床V1为例，下列填写内容为：
      # fileName: file
      # tokenName: Authorization
      # api: https://xxxxxx/api/v1/upload
      # token: Bearer xxxxxxxxxxxxxx
      # resp: data.links.url
      # 以兰空图床V2为例，下列填写内容为：
      # fileName: image
      # tokenName: token
      # api: https://xxxxxx/api/upload
      # token: xxxxxxxxxxxxxx
      # resp: data.url
    #   fileName: # 根据版本二选一
    #   tokenName: # 根据版本二选一
    #   api: # 图床 api 地址
    #   token: # 图床验证
    #   resp: # 图片地址返回值的字段
```
