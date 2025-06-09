---
title: Hexo-Stellar主题装修日记(二)
categories: [装修日记]
tags: [主题装修]
---

## 给超长代码块增加滚动条[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#给超长代码块增加滚动条)

首先判断代码块是否过长，如果是，则设置最大高度并开启滚动。

新建 `source/js/adjust-codeblock-height.js`，添加以下内容：

adjust-code-block-height.js

```
document.addEventListener("DOMContentLoaded", function() {

    // 选择所有的.md-text元素

    var codeBlocks = document.querySelectorAll('.md-text');

    // 遍历每个.md-text元素

    codeBlocks.forEach(function(block) {

      // 检查是否包含.highlight类的子元素，且父元素高度超过500px

      var highlightBlocks = block.querySelectorAll('.highlight');

      highlightBlocks.forEach(function(highlightBlock) {

        if (highlightBlock.clientHeight > 800) {

          highlightBlock.style.maxHeight = '300px';

          highlightBlock.style.overflow = 'auto';

        }

      });

    });

  });
```

以上代码代表如果代码框高度超过 800px，则开启折叠，折叠框最大高度为 300px。其中，可自行设置判断阈值 `if (highlightBlock.clientHeight > 800) {` 以及折叠后最大高度 `highlightBlock.style.maxHeight = '300px';`。

## 雪花特效[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#雪花特效)

代码来自[这里](https://mengru.space/?posts/2021/12/嗖得一下11月##博客下雪的小脚本)。我稍微做了一点修改，做成了一个按钮引入到主题中并用 localStorage 记录下雪状态，很简单的代码完美的解决了我的强迫症～

## 博客已运行x天x小时x分钟[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#博客已运行x天x小时x分钟)

在网站页脚部分添加一个“博客已运行 x 天 x 小时 x 分钟”字样，显示效果：

![勉强运行x天x小时x分钟x秒](Hexo-Stellar主题装修二/CleanShot 2024-04-16 at 21.51.07@2x-1749438579838-1030.webp)勉强运行x天x小时x分钟x秒



代码抄自[这里](https://blog.bxzdyg.cn/p/使用Hexo和Stellar搭建个人博客网站/#站点统计)，我为了调整样式加了一行代码 ![blobcat:blobcatpeekaboo](Hexo-Stellar主题装修二/blobcatpeekaboo-1749438579838-1032.png)。在 `_config.stellar.yml` 里添加如下代码，其中 `<span class='runtime'>` 中的类名 `runtime` 可自行设置。

```
footer:

  ...

  content: | # 支持 Markdown 格式

      <span id="runtime_span"></span>

      <script type="text/javascript">

      function show_runtime() {

          window.setTimeout("show_runtime()", 1000);

          X = new Date("2024/01/01 17:00:00"); // 网站开始运行的日期和时间

          Y = new Date(); // 当前日期和时间

          T = (Y.getTime() - X.getTime()); // 网站运行的总毫秒数

          M = 24 * 60 * 60 * 1000; // 一天的毫秒数

          a = T / M; // 总天数

          A = Math.floor(a); // 总天数的整数部分

          b = (a - A) * 24; // 总小时数

          B = Math.floor(b); // 总小时数的整数部分

          c = (b - B) * 60; // 总分钟数

          C = Math.floor((b - B) * 60); // 总分钟数的整数部分

          D = Math.floor((c - C) * 60); // 总秒数

          runtime_span.innerHTML = "⏱️勉强运行 <span class='runtime'>" + A + "天" + B + "小时" + C + "分" + D + "秒</span>";

      }

      show_runtime();

      </script>
```

再在自定义的 css 文件里添加以下代码，其中 color 可设置为主题色 `var(--theme-link)` 或自行设置：

```
.runtime

{

    font-weight: bold;

    color: #7F84A7;

}
```

## 页脚增加猫猫图片[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#页脚增加猫猫图片)

显示效果：

![img](Hexo-Stellar主题装修二/CleanShot 2024-04-17 at 19.09.44@2x-1749438579838-1034.webp)

首先，如果是使用本地图片，将图片上传到主题的资源文件夹，比如 `source/asset/posts/keyboard.png`

然后在主题配置文件的 `_config.stellar.yml` 中添加：

```
footer:

  ...

  content: | # 支持 Markdown 格式

  <img src="/你的/图片/路径.png" alt="描述文字" style="float: right; width: 60px; margin-left: 20px;">
```

其中 `float: right` 限定图片右对齐，`width:60px` 限制图片大小，可自行调整。

## 外部链接后面显示图标[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#外部链接后面显示图标)

显示效果：



![外部链接图标](Hexo-Stellar主题装修二/CleanShot 2024-04-16 at 22.09.38@2x-1749438579838-1038.webp)外部链接图标



方法一：



WARNING

老方法依赖 cheerio 模块，可行，但似乎会带来一些网站加载过慢的问题，我现在已经开心地转用新方法了，把老方法摆在这里全当（水）记（字）录（数）。

新建 `themes/stellar/scripts/filters/link-icon.js` 文件，增加以下代码：

```
//使用 cheerio 模块在文章中的外部链接后添加一个小图标：npm i cheerio --savehexo.extend.filter.register('after_render:html', function(html, data) {    const cheerio = require('cheerio');    const $ = cheerio.load(html, {decodeEntities: false});
    // 只选择<article class="md-text content">元素内的<a>标签    $('article.md-text.content a, footer.page-footer.footnote a').each(function() {      const link = $(this);      const href = link.attr('href');    //排除一些特殊的链接    if (!link.parents('div.tag-plugin.users-wrap').length && !link.parents('div.tag-plugin.sites-wrap').length && !link.parent('div.tag-plugin.ghcard').length && !link.parents('div.tag-plugin.link.dis-select').length && !link.parents('div.tag-plugin.colorful.note').length && !link.parents('div.social-wrap.dis-select').length) {
      // 确保链接的 href 属性存在，并检查其是否以 'http' 或 '/' 开头      if (href && (href.startsWith('http') || href.startsWith('/'))) {        link.html(link.html() + ` <span style="white-space: nowrap;"><svg width=".7em" height=".7em" viewBox="0 0 21 21" xmlns="http://www.w3.org/2000/svg"><path d="m13 3l3.293 3.293l-7 7l1.414 1.414l7-7L21 11V3z" fill="currentColor" /><path d="M19 19H5V5h7l-2-2H5c-1.103 0-2 .897-2 2v14c0 1.103.897 2 2 2h14c1.103 0 2-.897 2-2v-5l-2-2v7z" fill="currentColor"></svg></span>`);        //link.attr('target', '_blank'); // 可选：确保链接在新标签页打开      }    }    });
    return $.html();  });
```

方法二：



TIP

用老方法配置完我总觉得使用 Cheerio 模块后会导致网站加载过慢，就又优化了一下。询问 ChatGPT 得知可以考虑不使用 Node.js 的服务器端处理，而是使用纯前端的方法来达到同样的效果，通过在客户端 JavaScript 中添加代码来实现类似的功能，而不是在 Hexo 的后端渲染过程中处理。（好了，可以卸载 cheerio 了）

下面的这段代码可以在页面加载完成后运行，它会查找指定元素中的链接，并在这些链接后添加一个图标。这种方法的好处是，它不需要服务端的处理，所有操作都在用户的浏览器内完成，可以减少服务器负担，并且避免可能因服务器端渲染引起的加载问题。此外，这种方法也提供了更好的用户体验，因为它不会延迟页面内容的显示。

新建`source/js/link-icon.js` 文件，填入以下内容：

```
document.addEventListener('DOMContentLoaded', function () {    console.log('Document is ready.');
    const links = document.querySelectorAll('article.md-text.content a, footer.page-footer.footnote a');    console.log('Links found:', links.length);
    links.forEach(function(link) {        console.log('Processing link:', link.href);
        const parentClasses = ['tag-plugin.users-wrap', 'tag-plugin.sites-wrap', 'tag-plugin.ghcard', 'tag-plugin.link.dis-select', 'tag-plugin.colorful.note', 'social-wrap.dis-select'];        let skip = false;
        parentClasses.forEach(pc => {            if (link.closest(`div.${pc}`)) {                skip = true;                console.log('Skipping link due to parent class:', pc);            }        });
        if (!skip) {            const href = link.getAttribute('href');            console.log('Link href:', href);
            if (href && (href.startsWith('http') || href.startsWith('/'))) {                link.innerHTML += ` <span style="white-space: nowrap;"><svg width=".7em" height=".7em" viewBox="0 0 21 21" xmlns="http://www.w3.org/2000/svg"><path d="m13 3l3.293 3.293l-7 7l1.414 1.414l7-7L21 11V3z" fill="currentColor" /><path d="M19 19H5V5h7l-2-2H5c-1.103 0-2 .897-2 2v14c0 1.103.897 2 2 2h14c1.103 0 2-.897 2-2v-5l-2-2v7z" fill="currentColor"></svg></span>`;                console.log('Icon added to link:', link.innerHTML);            }        }    });});
```

这里做了两个筛选：

1. `const parentClasses = ['tag-plugin.users-wrap', 'tag-plugin.sites-wrap', 'tag-plugin.ghcard', 'tag-plugin.link.dis-select', 'tag-plugin.colorful.note', 'social-wrap.dis-select'];` 是被排除的类，可自行增减；
2. `if (href && (href.startsWith('http') || href.startsWith('/')))` 判断链接是否以 `http` 或 `/` 开头，如果不想给站内链接添加图标的话可以把后面的筛选条件去掉。

然后在主题文件 `_config.stellar.yml` 中引入：

```
inject:

  head:

  ...

    - <script src="/js/link-icon.js"></script> # 链接图标
```

## 增加参与讨论按钮[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#增加参与讨论按钮)

代码抄自[星日语](https://weekdaycare.cn/)，最新主题已自带此功能。

## 适配 Obsidian Callouts 标注块语法[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#适配-obsidian-callouts-标注块语法)

显示效果：

![img](Hexo-Stellar主题装修二/CleanShot 2024-04-16 at 22.23.51@2x-1749438579838-1036.webp)

暗黑模式下的显示效果：

![img](Hexo-Stellar主题装修二/CleanShot 2024-04-16 at 22.24.19@2x-1749438579839-1040.webp)

参考了 [Hexo 博客适配 Obsidian 新语法](https://uuanqin.top/p/d4bc55f2/index.html)，基础的设置请参考此链接。我暂时用不上其他功能，就把 callout 的样式搬来并做了一些修改。我个人还挺喜欢这个 callout 样式，比 quote 要好看而且添加也很方便，主要是可以和 Obsidian 打通，嘿嘿。

### 样式修改[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#样式修改)

原版的 callouts 标注块样式间距太大，我在此基础上改了 callout_blocks_common.css（不是很懂，写得很烂……但是能用）：

```
:root{--callout-note:68,138,255;--callout-abstract:0,176,255;--callout-info:0,184,212;--callout-tip:0,191,165;--callout-success:8,185,78;--callout-question:224,172,0;--callout-warning:255,145,0;--callout-failure:255,82,82;--callout-danger:255,23,68;--callout-bug:245,0,87;--callout-example:124,77,255;--callout-quote:158,158,158;--callout-radius:6px;--callout-border-opacity:0.5;--callout-title-bg-opacity:0.08}.callout-fold:before{align-self:center;content:url('data:image/svg+xml;charset=utf-8,<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="gray" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="chevron-down"><path d="m6 9 6 6 6-6"/></svg>')}.callout-fold{display:flex;transform:rotate(-90deg);transition:.5s cubic-bezier(.075,.82,.165,1)}.custom-callout[open]>summary>.callout-fold{transform:rotate(0deg)}.custom-callout>summary{border-top-left-radius:var(--callout-radius);border-top-right-radius:var(--callout-radius);cursor:pointer;margin:0;padding:0.5rem 1rem}.custom-callout>summary::marker{content:""}.custom-callout>summary:before{margin-right:.5rem}.custom-callout>summary::-webkit-details-marker{display:none}.callout-title{--fsp: calc(17px - 1px);font-size: var(--fsp);display:flex;justify-content:space-between;font-weight:bold;}.custom-callout>.callout-body{background:transparent!important;border-left:none;margin:0!important;padding:.3rem 1rem;position:relative}
.custom-callout>.callout-body>p{--fsp: calc(17px - 1px);font-size: var(--fsp);margin:8px 0}.custom-callout>.callout-body>pre{margin:1.25rem -1rem}.custom-callout>.callout-body>pre:first-child{margin-top:-.75rem}.custom-callout>.callout-body>pre:last-child{margin-bottom:-.75rem}
.custom-callout.note,.custom-callout.seealso{border-color:rgba(var(--callout-note),var(--callout-border-opacity))}.custom-callout.note>summary,.custom-callout.seealso>summary{    background-color:rgba(var(--callout-note),var(--callout-title-bg-opacity));    color:rgba(var(var(--callout-note)))}
.custom-callout.abstract,.custom-callout.summary,.custom-callout.tldr{border-color:rgba(var(--callout-abstract),var(--callout-border-opacity))}.custom-callout.abstract>summary,.custom-callout.summary>summary,.custom-callout.tldr>summary{    background-color:rgba(var(--callout-abstract),var(--callout-title-bg-opacity));    color:rgba(var(--callout-abstract))}
.custom-callout.info,.custom-callout.todo{border-color:rgba(var(--callout-info),var(--callout-border-opacity))}.custom-callout.info>summary,.custom-callout.todo>summary{    background-color:rgba(var(--callout-info),var(--callout-title-bg-opacity));    color:rgba(var(--callout-info))}

.custom-callout.hint,.custom-callout.important,.custom-callout.tip{border-color:rgba(var(--callout-tip),var(--callout-border-opacity))}.custom-callout.hint>summary,.custom-callout.important>summary,.custom-callout.tip>summary{    background-color:rgba(var(--callout-tip),var(--callout-title-bg-opacity));    color:rgba(var(--callout-tip))}
.custom-callout.check,.custom-callout.done,.custom-callout.success{border-color:rgba(var(--callout-success),var(--callout-border-opacity))}.custom-callout.check>summary,.custom-callout.done>summary,.custom-callout.success>summary{    background-color:rgba(var(--callout-success),var(--callout-title-bg-opacity));    color:rgba(var(--callout-success))}
.custom-callout.faq,.custom-callout.help,.custom-callout.question{border-color:rgba(var(--callout-question),var(--callout-border-opacity))}.custom-callout.faq>summary,.custom-callout.help>summary,.custom-callout.question>summary{    background-color:rgba(var(--callout-question),var(--callout-title-bg-opacity));    color:rgba(var(--callout-question))}
.custom-callout.attention,.custom-callout.caution,.custom-callout.warning{border-color:rgba(var(--callout-warning),var(--callout-border-opacity))}.custom-callout.attention>summary,.custom-callout.caution>summary,.custom-callout.warning>summary{    background-color:rgba(var(--callout-warning),var(--callout-title-bg-opacity));    color:rgba(var(--callout-warning))}
.custom-callout.fail,.custom-callout.failure,.custom-callout.missing{border-color:rgba(var(--callout-failure),var(--callout-border-opacity))}.custom-callout.fail>summary,.custom-callout.failure>summary,.custom-callout.missing>summary{    background-color:rgba(var(--callout-failure),var(--callout-title-bg-opacity));    color:rgba(var(--callout-failure))}
.custom-callout.danger,.custom-callout.error{border-color:rgba(var(--callout-danger),var(--callout-border-opacity))}.custom-callout.danger>summary,.custom-callout.error>summary{    background-color:rgba(var(--callout-danger),var(--callout-title-bg-opacity));    color:rgba(var(--callout-danger))}
.custom-callout.bug{border-color:rgba(var(--callout-bug),var(--callout-border-opacity))}.custom-callout.bug>summary{    background-color:rgba(var(--callout-bug),var(--callout-title-bg-opacity));    color:rgba(var(--callout-bug))}
.custom-callout.example{border-color:rgba(var(--callout-example),var(--callout-border-opacity))}.custom-callout.example>summary{    background-color:rgba(var(--callout-example),var(--callout-title-bg-opacity));    color:rgba(var(--callout-example))}
.custom-callout.cite,.custom-callout.quote{border-color:rgba(var(--callout-quote),var(--callout-border-opacity))}.custom-callout.cite>summary,.custom-callout.quote>summary{    background-color:rgba(var(--callout-quote),var(--callout-title-bg-opacity));    color:rgba(var(--callout-quote))}
.callout-title>.callout-icon+div{-webkit-box-flex:1;-ms-flex:1 1 0%;-webkit-flex:1 1 0%;flex:1 1 0%;margin-left:.25rem}.callout-icon{align-items:center;color:#000;display:flex}.callout-icon:before{height:20px;width:20px}.custom-callout.attention>.callout-title>.callout-icon:before,.custom-callout.caution>.callout-title>.callout-icon:before,.custom-callout.warning>.callout-title>.callout-icon:before{content:url('data:image/svg+xml;charset=utf-8,<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="%23FF9100" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-alert-triangle"><path d="m21.73 18-8-14a2 2 0 0 0-3.48 0l-8 14A2 2 0 0 0 4 21h16a2 2 0 0 0 1.73-3ZM12 9v4M12 17h.01"/></svg>')}.custom-callout.note>.callout-title>.callout-icon:before,.custom-callout.seealso>.callout-title>.callout-icon:before{content:url('data:image/svg+xml;charset=utf-8,<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="%23448AFF" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-pencil"><path d="m18 2 4 4M7.5 20.5 19 9l-4-4L3.5 16.5 2 22z"/></svg>')}.custom-callout.abstract>.callout-title>.callout-icon:before,.custom-callout.summary>.callout-title>.callout-icon:before,.custom-callout.tldr>.callout-title>.callout-icon:before{content:url('data:image/svg+xml;charset=utf-8,<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="%2300B0FF" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-clipboard-list"><rect x="8" y="2" width="8" height="4" rx="1" ry="1"/><path d="M16 4h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2M12 11h4M12 16h4M8 11h.01M8 16h.01"/></svg>')}.custom-callout.info>.callout-title>.callout-icon:before,.custom-callout.todo>.callout-title>.callout-icon:before{content:url('data:image/svg+xml;charset=utf-8,<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="%2300B8D4" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-check-circle-2"><path d="M12 22c5.523 0 10-4.477 10-10S17.523 2 12 2 2 6.477 2 12s4.477 10 10 10z"/><path d="m9 12 2 2 4-4"/></svg>')}.custom-callout.hint>.callout-title>.callout-icon:before,.custom-callout.important>.callout-title>.callout-icon:before,.custom-callout.tip>.callout-title>.callout-icon:before{content:url('data:image/svg+xml;charset=utf-8,<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="%2300BFA5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-flame"><path d="M8.5 14.5A2.5 2.5 0 0 0 11 12c0-1.38-.5-2-1-3-1.072-2.143-.224-4.054 2-6 .5 2.5 2 4.9 4 6.5 2 1.6 3 3.5 3 5.5a7 7 0 1 1-14 0c0-1.153.433-2.294 1-3a2.5 2.5 0 0 0 2.5 2.5z"/></svg>')}.custom-callout.check>.callout-title>.callout-icon:before,.custom-callout.done>.callout-title>.callout-icon:before,.custom-callout.success>.callout-title>.callout-icon:before{content:url('data:image/svg+xml;charset=utf-8,<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="%2300C853" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-check"><path d="M20 6 9 17l-5-5"/></svg>')}.custom-callout.faq>.callout-title>.callout-icon:before,.custom-callout.help>.callout-title>.callout-icon:before,.custom-callout.question>.callout-title>.callout-icon:before{content:url('data:image/svg+xml;charset=utf-8,<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="%23E0AC00" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-help-circle"><circle cx="12" cy="12" r="10"/><path d="M9.09 9a3 3 0 0 1 5.83 1c0 2-3 3-3 3M12 17h.01"/></svg>')}.custom-callout.fail>.callout-title>.callout-icon:before,.custom-callout.failure>.callout-title>.callout-icon:before,.custom-callout.missing>.callout-title>.callout-icon:before{content:url('data:image/svg+xml;charset=utf-8,<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="%23FF5252" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-x"><path d="M18 6 6 18M6 6l12 12"/></svg>')}.custom-callout.danger>.callout-title>.callout-icon:before,.custom-callout.error>.callout-title>.callout-icon:before{content:url('data:image/svg+xml;charset=utf-8,<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="%23FF1744" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-zap"><path d="M13 2 3 14h9l-1 8 10-12h-9l1-8z"/></svg>')}.custom-callout.bug>.callout-title>.callout-icon:before{content:url('data:image/svg+xml;charset=utf-8,<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="%23F50057" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-bug"><rect x="8" y="6" width="8" height="14" rx="4"/><path d="m19 7-3 2M5 7l3 2M19 19l-3-2M5 19l3-2M20 13h-4M4 13h4M10 4l1 2M14 4l-1 2"/></svg>')}.custom-callout.example>.callout-title>.callout-icon:before{content:url('data:image/svg+xml;charset=utf-8,<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="%237C4DFF" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-list"><path d="M8 6h13M8 12h13M8 18h13M3 6h.01M3 12h.01M3 18h.01"/></svg>')}.custom-callout.cite>.callout-title>.callout-icon:before,.custom-callout.quote>.callout-title>.callout-icon:before{content:url('data:image/svg+xml;charset=utf-8,<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="%239E9E9E" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-quote"><path d="M3 21c3 0 7-1 7-8V5c0-1.25-.756-2.017-2-2H4c-1.25 0-2 .75-2 1.972V11c0 1.25.75 2 2 2 1 0 1 0 1 1v1c0 1-1 2-2 2s-1 .008-1 1.031V20c0 1 0 1 1 1zM15 21c3 0 7-1 7-8V5c0-1.25-.757-2.017-2-2h-4c-1.25 0-2 .75-2 1.972V11c0 1.25.75 2 2 2h.75c0 2.25.25 4-2.75 4v3c0 1 0 1 1 1z"/></svg>')}

.custom-callout.note > .callout-body {    /* 移除了 background:transparent!important; 改为根据类型变化的背景色 */    background-color: rgba(var(--callout-note), var(--callout-title-bg-opacity));  }
  /* 根据不同的类型设置背景色和文字/图标颜色 */.custom-callout.note, .custom-callout.note > summary {    background-color: rgba(var(--callout-note), var(--callout-title-bg-opacity));  }
.custom-callout.abstract > .callout-body {    background-color: rgba(var(--callout-abstract), var(--callout-title-bg-opacity));  }
.custom-callout.abstract, .custom-callout.abstract > summary {    background-color: rgba(var(--callout-abstract), var(--callout-title-bg-opacity));  }
.custom-callout.info > .callout-body {    background-color: rgba(var(--callout-info), var(--callout-title-bg-opacity));  }
.custom-callout.info, .custom-callout.info > summary {    background-color: rgba(var(--callout-info), var(--callout-title-bg-opacity));  }
.custom-callout.tip > .callout-body {    background-color: rgba(var(--callout-tip), var(--callout-title-bg-opacity));  }
.custom-callout.tip, .custom-callout.tip > summary {    background-color: rgba(var(--callout-tip), var(--callout-title-bg-opacity));  }
.custom-callout.success > .callout-body {    background-color: rgba(var(--callout-success), var(--callout-title-bg-opacity));  }
.custom-callout.success, .custom-callout.success > summary {    background-color: rgba(var(--callout-success), var(--callout-title-bg-opacity));  }
.custom-callout.question > .callout-body {    background-color: rgba(var(--callout-question), var(--callout-title-bg-opacity));  }
.custom-callout.question, .custom-callout.question > summary {    background-color: rgba(var(--callout-question), var(--callout-title-bg-opacity));  }
.custom-callout.warning > .callout-body {    background-color: rgba(var(--callout-warning), var(--callout-title-bg-opacity));  }
.custom-callout.warning, .custom-callout.warning > summary {    background-color: rgba(var(--callout-warning), var(--callout-title-bg-opacity));  }
.custom-callout.failure > .callout-body {    background-color: rgba(var(--callout-failure), var(--callout-title-bg-opacity));  }
.custom-callout.failure, .custom-callout.failure > summary {    background-color: rgba(var(--callout-failure), var(--callout-title-bg-opacity));  }
.custom-callout.danger > .callout-body {    background-color: rgba(var(--callout-danger), var(--callout-title-bg-opacity));  }
.custom-callout.danger, .custom-callout.danger > summary {    background-color: rgba(var(--callout-danger), var(--callout-title-bg-opacity));  }
.custom-callout.bug > .callout-body {    background-color: rgba(var(--callout-bug), var(--callout-title-bg-opacity));  }
.custom-callout.bug, .custom-callout.bug > summary {    background-color: rgba(var(--callout-bug), var(--callout-title-bg-opacity));  }
.custom-callout.example > .callout-body {    background-color: rgba(var(--callout-example), var(--callout-title-bg-opacity));  }
.custom-callout.example, .custom-callout.example > summary {    background-color: rgba(var(--callout-example), var(--callout-title-bg-opacity));  }
.custom-callout.quote > .callout-body {    background-color: rgba(var(--callout-quote), var(--callout-title-bg-opacity));  }
.custom-callout.quote, .custom-callout.quote > summary {    background-color: rgba(var(--callout-quote), var(--callout-title-bg-opacity));  }
.custom-callout.cite > .callout-body {    background-color: rgba(var(--callout-quote), var(--callout-title-bg-opacity));  }
.custom-callout.cite, .custom-callout.cite > summary {    background-color: rgba(var(--callout-quote), var(--callout-title-bg-opacity));  }
.custom-callout.todo > .callout-body {    background-color: rgba(var(--callout-info), var(--callout-title-bg-opacity));  }
.custom-callout.todo, .custom-callout.todo > summary {    background-color: rgba(var(--callout-info), var(--callout-title-bg-opacity));  }
.custom-callout.seealso > .callout-body {    background-color: rgba(var(--callout-note), var(--callout-title-bg-opacity));  }
.custom-callout.seealso, .custom-callout.seealso > summary {    background-color: rgba(var(--callout-note), var(--callout-title-bg-opacity));  }
.custom-callout.hint > .callout-body {    background-color: rgba(var(--callout-tip), var(--callout-title-bg-opacity));  }
.custom-callout.hint, .custom-callout.hint > summary {    background-color: rgba(var(--callout-tip), var(--callout-title-bg-opacity));  }
.custom-callout.important > .callout-body {    background-color: rgba(var(--callout-tip), var(--callout-title-bg-opacity));  }
.custom-callout.important, .custom-callout.important > summary {    background-color: rgba(var(--callout-tip), var(--callout-title-bg-opacity));  }
.custom-callout.attention > .callout-body {    background-color: rgba(var(--callout-warning), var(--callout-title-bg-opacity));  }
.custom-callout.attention, .custom-callout.attention > summary {    background-color: rgba(var(--callout-warning), var(--callout-title-bg-opacity));  }
.custom-callout.caution > .callout-body {    background-color: rgba(var(--callout-warning), var(--callout-title-bg-opacity));  }
.custom-callout.caution, .custom-callout.caution > summary {    background-color: rgba(var(--callout-warning), var(--callout-title-bg-opacity));  }
.custom-callout.done > .callout-body {    background-color: rgba(var(--callout-success), var(--callout-title-bg-opacity));  }
.custom-callout.done, .custom-callout.done > summary {    background-color: rgba(var(--callout-success), var(--callout-title-bg-opacity));  }
.custom-callout.check > .callout-body {    background-color: rgba(var(--callout-success), var(--callout-title-bg-opacity));  }
.custom-callout.check, .custom-callout.check > summary {    background-color: rgba(var(--callout-success), var(--callout-title-bg-opacity));  }
.custom-callout.faq > .callout-body {    background-color: rgba(var(--callout-question), var(--callout-title-bg-opacity));  }
.custom-callout.faq, .custom-callout.faq > summary {    background-color: rgba(var(--callout-question), var(--callout-title-bg-opacity));  }
.custom-callout.help > .callout-body {    background-color: rgba(var(--callout-question), var(--callout-title-bg-opacity));  }
.custom-callout.help, .custom-callout.help > summary {    background-color: rgba(var(--callout-question), var(--callout-title-bg-opacity));  }
.custom-callout.fail > .callout-body {    background-color: rgba(var(--callout-failure), var(--callout-title-bg-opacity));  }
.custom-callout.fail, .custom-callout.fail > summary {    background-color: rgba(var(--callout-failure), var(--callout-title-bg-opacity));  }
.custom-callout.missing > .callout-body {    background-color: rgba(var(--callout-failure), var(--callout-title-bg-opacity));  }
.custom-callout.missing, .custom-callout.missing > summary {    background-color: rgba(var(--callout-failure), var(--callout-title-bg-opacity));  }
.custom-callout.error > .callout-body {    background-color: rgba(var(--callout-danger), var(--callout-title-bg-opacity));  }
.custom-callout.error, .custom-callout.error > summary {    background-color: rgba(var(--callout-danger), var(--callout-title-bg-opacity));  }
.custom-callout.tldr > .callout-body {    background-color: rgba(var(--callout-abstract), var(--callout-title-bg-opacity));  }
.custom-callout.tldr, .custom-callout.tldr > summary {    background-color: rgba(var(--callout-abstract), var(--callout-title-bg-opacity));  }
```

## 集成 Telegram Channel 说说[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#集成-telegram-channel-说说)

显示效果：

篇幅限制，只展示2条，请耐心等待加载。（可能要挂代理）

代码抄自[把Tg Channel接入到Stellar时间线](https://blog.hzchu.top/2024/把Tg-Channel接入到Stellar时间线/)。因为我懒得做标签筛选所以直接把这个去掉啦，在此还要感谢佬的耐心解答 ![blobcat:ablobcatheart](Hexo-Stellar主题装修二/ablobcatheart-1749438579839-1042.png)

## GitHub Action 自动部署并修复更新时间[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#github-action-自动部署并修复更新时间)

在自动部署这里遇到了几个坑，总结下来大概有下：

- 网上流行的很多 yml workflow 文件都有些过时
- 公钥私钥啥的不太懂，配置了半天
- 因为我的博客有数学公式显示，所以要在 workflow 里加入安装 pandoc 的部分，才能够成功运行
- 自动部署后网站的文章更新时间全部变成 push 时间，但在本地是正常的。一番搜索后找到了解决方法，在 yml 文件里加入了以下代码，分别修复 posts、wiki、notes 的更新时间：

```
  - name: Restore file modification time 🕒      run: find source/_posts -name '*.md' | while read file; do touch -d "$(git log -1 --format="@%ct" "$file")" "$file"; done
    - name: Restore file modification time of wiki🕒      run: find source/wiki -name '*.md' | while read file; do touch -d "$(git log -1 --format="@%ct" "$file")" "$file"; done
    - name: Restore file modification time of notes🕒      run: find source/notes -name '*.md' | while read file; do touch -d "$(git log -1 --format="@%ct" "$file")" "$file"; done
```

最后附上完整代码，拿去用的话要自己配置一下 GitHub 部分的设置：

```
name: auto deploy
on:  workflow_dispatch:  push:
jobs:  build:    runs-on: ubuntu-latest # 运行环境为最新版 Ubuntu    name: auto deploy    steps:    # 1. 获取源码    - name: Checkout      uses: actions/checkout@v4 # 使用 actions/checkout@v3      with: # 条件        submodules: true # Checkout private submodules(themes or something else). 当有子模块时切换分支？        fetch-depth: 0
    - name: Restore file modification time 🕒      run: find source/_posts -name '*.md' | while read file; do touch -d "$(git log -1 --format="@%ct" "$file")" "$file"; done
    - name: Restore file modification time of wiki🕒      run: find source/wiki -name '*.md' | while read file; do touch -d "$(git log -1 --format="@%ct" "$file")" "$file"; done
    - name: Restore file modification time of notes🕒      run: find source/notes -name '*.md' | while read file; do touch -d "$(git log -1 --format="@%ct" "$file")" "$file"; done
    # 2. 配置环境    - name: Setup Node.js 18.19.x      uses: actions/setup-node@master      with:        node-version: "18.19.x"
    - name: Install pandoc      run: |        cd /tmp        wget -c https://github.com/jgm/pandoc/releases/download/2.14.0.3/pandoc-2.14.0.3-1-amd64.deb        sudo dpkg -i pandoc-2.14.0.3-1-amd64.deb
    # 3. 生成静态文件    - name: Generate Public Files      run: |        npm i        npm install hexo-cli -g        hexo clean && hexo generate    # 4a. 部署到 GitHub 仓库（可选）    - name: Deploy to GitHub Pages      uses: peaceiris/actions-gh-pages@v3      with:        deploy_key: ${{ secrets.HEXO_DEPLOY_PRI }} # 配置密钥        external_repository: # 填入你的GitHub pages部署仓库        publish_branch: gt-pages # 填入部署分支        publish_dir: ./public        commit_message: ${{ github.event.head_commit.message }}        user_name: 'github-actions[bot]'        user_email: 'github-actions[bot]@users.noreply.github.com'
```

## 给博客添加地理定位并制作个性欢迎[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#给博客添加地理定位并制作个性欢迎)

显示效果：

![个性欢迎卡片](Hexo-Stellar主题装修二/CleanShot 2024-04-17 at 01.09.07@2x-1749438579839-1044.webp)个性欢迎卡片



代码来自[给博客添加腾讯地图定位并制作个性欢迎](https://ichika.cc/Article/beautiful_IPLocation/)。我稍微做了一点调整：

新建 `source/js/services/txmap.js`，并添加以下代码： {% folding 点击展开代码 %}

```
//get请求$.ajax({    type: 'get',    url: 'https://apis.map.qq.com/ws/location/v1/ip',    data: {        key: '你的key',        output: 'jsonp',    },    dataType: 'jsonp',    success: function (res) {        ipLoacation = res;    }})function getDistance(e1, n1, e2, n2) {    const R = 6371    const { sin, cos, asin, PI, hypot } = Math    let getPoint = (e, n) => {        e *= PI / 180        n *= PI / 180        return { x: cos(n) * cos(e), y: cos(n) * sin(e), z: sin(n) }    }
    let a = getPoint(e1, n1)    let b = getPoint(e2, n2)    let c = hypot(a.x - b.x, a.y - b.y, a.z - b.z)    let r = asin(c / 2) * 2 * R    return Math.round(r);}
function showWelcome() {
    let dist = getDistance(113.34499552, 23.15537143, ipLoacation.result.location.lng, ipLoacation.result.location.lat); //这里换成自己的经纬度    let pos = ipLoacation.result.ad_info.nation;    let ip;    let posdesc;    //根据国家、省份、城市信息自定义欢迎语    switch (ipLoacation.result.ad_info.nation) {        case "日本":            posdesc = "よろしく，一起去看樱花吗";            break;        case "美国":            posdesc = "Let us live in peace!";            break;        case "英国":            posdesc = "想同你一起夜乘伦敦眼";            break;        case "俄罗斯":            posdesc = "干了这瓶伏特加！";            break;        case "法国":            posdesc = "C'est La Vie";            break;        case "德国":            posdesc = "Die Zeit verging im Fluge.";            break;        case "澳大利亚":            posdesc = "一起去大堡礁吧！";            break;        case "加拿大":            posdesc = "拾起一片枫叶赠予你";            break;        case "中国":            pos = ipLoacation.result.ad_info.province + " " + ipLoacation.result.ad_info.city + " " + ipLoacation.result.ad_info.district;            ip = ipLoacation.result.ip;            switch (ipLoacation.result.ad_info.province) {                case "北京市":                    posdesc = "北——京——欢迎你~~~";                    break;                case "天津市":                    posdesc = "讲段相声吧。";                    break;                case "河北省":                    posdesc = "山势巍巍成壁垒，天下雄关。铁马金戈由此向，无限江山。";                    break;                case "山西省":                    posdesc = "展开坐具长三尺，已占山河五百余。";                    break;                case "内蒙古自治区":                    posdesc = "天苍苍，野茫茫，风吹草低见牛羊。";                    break;                case "辽宁省":                    posdesc = "我想吃烤鸡架！";                    break;                case "吉林省":                    posdesc = "状元阁就是东北烧烤之王。";                    break;                case "黑龙江省":                    posdesc = "很喜欢哈尔滨大剧院。";                    break;                case "上海市":                    posdesc = "众所周知，中国只有两个城市。";                    break;                case "江苏省":                    switch (ipLoacation.result.ad_info.city) {                        case "南京市":                            posdesc = "这是我挺想去的城市啦。";                            break;                        case "苏州市":                            posdesc = "上有天堂，下有苏杭。";                            break;                        default:                            posdesc = "散装是必须要散装的。";                            break;                    }                    break;                case "浙江省":                    posdesc = "东风渐绿西湖柳，雁已还人未南归。";                    break;                case "河南省":                    switch (ipLoacation.result.ad_info.city) {                        case "郑州市":                            posdesc = "豫州之域，天地之中。";                            break;                        case "南阳市":                            posdesc = "臣本布衣，躬耕于南阳。此南阳非彼南阳！";                            break;                        case "驻马店市":                            posdesc = "峰峰有奇石，石石挟仙气。嵖岈山的花很美哦！";                            break;                        case "开封市":                            posdesc = "刚正不阿包青天。";                            break;                        case "洛阳市":                            posdesc = "洛阳牡丹甲天下。";                            break;                        default:                            posdesc = "可否带我品尝河南烩面啦？";                            break;                    }                    break;                case "安徽省":                    posdesc = "蚌埠住了，芜湖起飞。";                    break;                case "福建省":                    posdesc = "井邑白云间，岩城远带山。";                    break;                case "江西省":                    posdesc = "落霞与孤鹜齐飞，秋水共长天一色。";                    break;                case "山东省":                    posdesc = "遥望齐州九点烟，一泓海水杯中泻。";                    break;                case "湖北省":                    posdesc = "来碗热干面！";                    break;                case "湖南省":                    posdesc = "74751，长沙斯塔克。";                    break;                case "广东省":                    posdesc = "老板来两斤福建人。";                    break;                case "广西壮族自治区":                    posdesc = "桂林山水甲天下。";                    break;                case "海南省":                    posdesc = "朝观日出逐白浪，夕看云起收霞光。";                    break;                case "四川省":                    posdesc = "康康川妹子。";                    break;                case "贵州省":                    posdesc = "茅台，学生，再塞200。";                    break;                case "云南省":                    posdesc = "玉龙飞舞云缠绕，万仞冰川直耸天。";                    break;                case "西藏自治区":                    posdesc = "躺在茫茫草原上，仰望蓝天。";                    break;                case "陕西省":                    posdesc = "来份臊子面加馍。";                    break;                case "甘肃省":                    posdesc = "羌笛何须怨杨柳，春风不度玉门关。";                    break;                case "青海省":                    posdesc = "牛肉干和老酸奶都好好吃。";                    break;                case "宁夏回族自治区":                    posdesc = "大漠孤烟直，长河落日圆。";                    break;                case "新疆维吾尔自治区":                    posdesc = "驼铃古道丝绸路，胡马犹闻唐汉风。";                    break;                case "台湾省":                    posdesc = "我在这头，大陆在那头。";                    break;                case "香港特别行政区":                    posdesc = "永定贼有残留地鬼嚎，迎击光非岁玉。";                    break;                case "澳门特别行政区":                    posdesc = "性感荷官，在线发牌。";                    break;                default:                    posdesc = "带我去你的城市逛逛吧！";                    break;            }            break;        default:            posdesc = "带我去你的国家逛逛吧。";            break;    }
    //根据本地时间切换欢迎语    let timeChange;    let date = new Date();    if (date.getHours() >= 5 && date.getHours() < 11) timeChange = "<span>上午好</span>，一日之计在于晨！";    else if (date.getHours() >= 11 && date.getHours() < 13) timeChange = "<span>中午好</span>，该摸鱼吃午饭了。";    else if (date.getHours() >= 13 && date.getHours() < 15) timeChange = "<span>下午好</span>，懒懒地睡个午觉吧！";    else if (date.getHours() >= 15 && date.getHours() < 16) timeChange = "<span>三点几啦</span>，一起饮茶呀！";    else if (date.getHours() >= 16 && date.getHours() < 19) timeChange = "<span>夕阳无限好！</span>";    else if (date.getHours() >= 19 && date.getHours() < 24) timeChange = "<span>晚上好</span>，夜生活嗨起来！";    else timeChange = "夜深了，早点休息，少熬夜。";
    try {        //自定义文本和需要放的位置        document.getElementById("welcome-info").innerHTML =            `<b><center>🎉 欢迎信息 🎉</center>&emsp;&emsp;欢迎来自 <span style="color:var(--theme-color)">${pos}</span> 的小伙伴，${timeChange}您现在距离站长约 <span style="color:var(--theme-color)">${dist}</span> 公里，当前的IP地址为： <span style="color:var(--theme-color)">${ip}</span>， ${posdesc}</b>`;    } catch (err) {        // console.log("Pjax无法获取#welcome-info元素🙄🙄🙄")    }}window.onload = showWelcome;// 如果使用了pjax在加上下面这行代码document.addEventListener('pjax:complete', showWelcome);
```

{% endfolding %}

### 在主题文件中配置[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#在主题文件中配置)

在主题配置文件 `_config.stellar.yml` 中引入jQuery依赖和刚刚的js文件：

```
inject:

    - <script src="https://cdn.staticfile.org/jquery/3.6.3/jquery.min.js"></script> # jQuery

  - <script async data-pjax src="/js/services/txmap.js"></script> # 腾讯位置API
```

在 `source/_data/widgets.yml` 中添加小组件，我在里面嵌套了一个随机文章跳转，不要的话可以删掉，**其中，`<span id="welcome-info" ></span>` 是必须的不可以删：**

```
welcomeloc:

  layout: markdown

  title: '🎉 抓到你啦'

  linklist:

    columns: 1

    items:

      - icon: '<img src="https://api.iconify.design/ion:dice-outline.svg"/>'

        title: 随机文章

        url: 'javascript:toRandomPost()'

  content: |

    <span id="welcome-info" style="font-family: LXGW WenKai Screen;"></span>
```

然后就跟正常的小组件一样在想要的地方引用即可。

## 添加更改字体按钮[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#添加更改字体按钮)

显示效果：

**第一种：** 在任意位置增加一个 button 按钮![img](Hexo-Stellar主题装修二/CleanShot 2024-04-17 at 01.21.07@2x-1749438579839-1046.webp)

鼠标放到上面会显示提示： ![img](Hexo-Stellar主题装修二/CleanShot 2024-04-17 at 01.22.42@2x-1749438579839-1048.webp)

**第二种：** 在文章页面目录下方显示

![img](Hexo-Stellar主题装修二/CleanShot 2024-04-17 at 01.24.07@2x-1749438579839-1050.webp)

之前一直纠结要不要把自定义字体效果去掉，在选择和留下之间来回切换 ![blobcat:ablobcatknitsweats](Hexo-Stellar主题装修二/ablobcatknitsweats-1749438579839-1052.png) 最终才出现了这里的方案：默认不加载任何字体，喜欢 LXGW 字体的话可点击图标转换，同时再点击一下就恢复。代码不长但完美地解决了我的强迫症～

### 第一步：准备字体文件[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#第一步准备字体文件)

可以是在线文件也可以是本地文件，我是在主题 config 文件下通过 inject 引入了 LXGW 字体。

### 第二步：修改 css[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#第二步修改-css)

首先确保 LXGW WenKai Screen 字体已经通过 CSS 正确引入。你可以在 CSS 文件中添加一个特定的类，用于当用户选择使用这种字体时切换到它：

```
/* 设置字体 */

.LXGWMode {

    font-family: 'LXGW WenKai Screen', system-ui, 'Helvetica Neue', sans-serif;  // 使用 LXGW WenKai 字体，并指定后备字体

  }
```

### 第三步：添加 javascript[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#第三步添加-javascript)

新建 `source/js/changefont.js` 文件，添加以下代码：

```
document.addEventListener('DOMContentLoaded', function () {    applyFontSetting();    updateButtonText(); // Ensure the button text is correct on page load});
document.addEventListener('pjax:success', function () {    applyFontSetting();    updateButtonText(); // Update the button text after PJAX updates});
function applyFontSetting() {    if (localStorage.getItem("LXGWFontEnabled") === "true") {        document.body.classList.add("LXGWMode");    } else {        document.body.classList.remove("LXGWMode");    }}
function toggleLXGWFont() {    var button = document.querySelector('.custom-button'); // Find the button    if (localStorage.getItem("LXGWFontEnabled") === "true") {        localStorage.setItem("LXGWFontEnabled", "false");        document.body.classList.remove("LXGWMode");        button.innerHTML = '<img src="https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/ablobcatrainbow.png" alt="Emoji" style="vertical-align: middle; width: 20px; height: 20px;"> 危险，请勿点击';    } else {        localStorage.setItem("LXGWFontEnabled", "true");        document.body.classList.add("LXGWMode");        button.innerHTML = '<img src="https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/ablobcatrainbow.png" alt="Emoji" style="vertical-align: middle; width: 20px; height: 20px;"> 不要说我没有警告过你';    }}
function updateButtonText() {    var button = document.querySelector('.custom-button'); // Find the button    if (localStorage.getItem("LXGWFontEnabled") === "true") {        button.innerHTML = '<img src="https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/ablobcatrainbow.png" alt="Emoji" style="vertical-align: middle; width: 20px; height: 20px;"> 不要点这里啦！';    } else {        button.innerHTML = '<img src="https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/ablobcatrainbow.png" alt="Emoji" style="vertical-align: middle; width: 20px; height: 20px;"> 危险，请勿点击';    }}
```

### 第四步：添加切换按钮[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#第四步添加切换按钮)

然后在想要的地方引用即可，可以自行添加各种 emoji，比如：

```
<button class="custom-button tooltip" onclick="toggleLXGWFont()" data-msg="警告，真的很危险"><img src="https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/ablobcatrainbow.png" alt="Emoji" style="vertical-align: middle; width: 20px; height: 20px;"> 危险，请勿点击</button>
```

### 给按钮加入 css 提示框[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#给按钮加入-css-提示框)

在自定义 css 文件中添加：

```
.custom-button {    display: inline-block;    padding: 2px 10px;    /*margin: 10px;    background-color: #f2f2f2; /* Light grey background, change as needed */    font-family: inherit; /* Inherits the font-family from parent container */    color: #835EEC;    background-color: #F2EEFD;    @media (prefers-color-scheme: dark) {      color: #A28BF2;      background-color: #282433;    }    text-align: center;    cursor: pointer;    /*border: 2px solid #ccc; /* Grey border */    border-radius: 16px; /* Rounded corners */    transition: all 0.3s ease;  }
.custom-button:hover {    background-color: #e9e9e9; /* Slightly darker on hover */    @media (prefers-color-scheme: dark) {      background-color: #333; /* Darker background on hover */    }    border-color: #999; /* Darker border on hover */  }
/* toggle-font 提示框的样式 */
.tooltip {    position: relative;    cursor: pointer; /* 可选，让用户知道这是一个可以互动的元素 */}
.tooltip:hover::before {    white-space: nowrap;    line-height: 18px;    content: attr(data-msg);    position: absolute;    padding: 0 8px;    display: block;    color: #ffffff;    background: #656565;    border-radius: 6px;    font-size: 12px;    top: -25px;    left: 50%;    transform: translateX(-50%);    Z-index: 1000; /* 确保提示框在其他元素之上 */}
.tooltip:hover:: after {    Content: "";    Position: absolute;    Top: -8 px;    Left: 50%;    Transform: translateX (-50%);    Border: 6 px solid transparent;    border-top-color: #656565 ; /* 简化写法 */}
/* toggle-font 按钮的样式 */
.widget-wrapper. Toggle-font {    Background: none; // Example: making background transparent    /* Add other styles specific to the toggle-font widget here */}
```

### 第二种样式[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#第二种样式)



WARNING

第二种样式需要对主题文件进行一丢丢修改，但貌似不太影响更新……只要无冲突的话可以一直 update fork

在 `languages/zh-CN.yml` 中添加一行 `font: 更改字体`，并在 icons.yml 里添加：

```
default:font: <svg class="theme-icon" xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 32 32"><path d="m12.677 17.781l-2.626-6.256l-2.694 6.256Zm6.723 6.511h-7.069v-1.365l.458-.023a1.847 1.847 0 0 0 .972-.2a.313.313 0 0 0 .145-.263a4.158 4.158 0 0 0-.419-1.4l-.812-1.931H7.322L6.4 21.259a3.319 3.319 0 0 0-.349 1.157c0 .036 0 .119.154.241a2.481 2.481 0 0 0 1.191.247l.448.033v1.354H2v-1.31l.4-.07a2.188 2.188 0 0 0 1-.318a6.318 6.318 0 0 0 1.18-2.066l5.575-13.036H11.2l5.512 13.174a5.255 5.255 0 0 0 1.049 1.835a1.959 1.959 0 0 0 1.19.4l.454.027Zm6.441-2.732v-3.985a22.542 22.542 0 0 0-2.226.97a3.845 3.845 0 0 0-1.29 1.05a2.03 2.03 0 0 0-.388 1.2a1.951 1.951 0 0 0 .491 1.362a1.49 1.49 0 0 0 1.13.544a4.142 4.142 0 0 0 2.283-1.141m-3.333 2.949a2.833 2.833 0 0 1-2.139-.893a3.206 3.206 0 0 1-.833-2.285a2.959 2.959 0 0 1 .415-1.577a5 5 0 0 1 1.791-1.625a23.876 23.876 0 0 1 3.617-1.588v-.074a2.905 2.905 0 0 0-.383-1.833a1.325 1.325 0 0 0-1.075-.412a1.155 1.155 0 0 0-.816.26a.687.687 0 0 0-.277.536l.023.646a1.62 1.62 0 0 1-.4 1.158a1.481 1.481 0 0 1-2.1-.019a1.634 1.634 0 0 1-.391-1.134a2.8 2.8 0 0 1 1.182-2.177a4.813 4.813 0 0 1 3.125-.932a5.381 5.381 0 0 1 2.508.524a2.628 2.628 0 0 1 1.213 1.346a6.391 6.391 0 0 1 .244 2.2v3.55a14.665 14.665 0 0 0 .051 1.749a.661.661 0 0 0 .054.2c.085-.078.284-.225.864-.806l.819-.828v1.967l-.1.128c-.958 1.283-1.883 1.907-2.83 1.907a1.6 1.6 0 0 1-1.257-.557a1.788 1.788 0 0 1-.358-.74a9.688 9.688 0 0 1-1.433.977a3.579 3.579 0 0 1-1.514.332"/></svg>
```

在 `layout/_partial/widgets/toc.ejs` 中，在想要的位置，如 `el += editBtn` 后，添加以下代码：

```
 el += `<a class="toggle-font" onclick="toggleLXGWFont()">`

  el += icon('default:font')

  el += `<span>${__('btn.font')}</span>`

  el += `</a>`
```

为了使这个图标随主题明暗自动变化，在自定义 css 文件中加入：

```
/* 设置图标颜色 *//* 白天模式，默认填充色为黑色 */.theme-icon {  fill: black;}
/* 暗黑模式，填充色为白色 */@media (prefers-color-scheme: dark) {  .theme-icon {      fill: white;  }}
```

## 随机文章跳转[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#随机文章跳转)



NOTE

要在主题文件夹里新增文件，不影响主题后续更新![blobcat:ablobcatattentionreverse](Hexo-Stellar主题装修二/ablobcatattentionreverse-1749438579839-1054.png)

终于来到了我最爱的生活哲学！代码参考了[这个链接](https://blog.zhheo.com/p/c116857c.html)。创建 `themes/stellar/scripts/helpers/random.js` ，增加以下代码：

```
hexo.extend.filter.register('after_render:html', function (data) {

  const posts = []

  hexo.locals.get('posts').map(function (post) {

    if (post.random !== false) posts.push(post.path)

  })

  data += `<script>var posts=${JSON.stringify(posts)};function toRandomPost(){ window.pjax ? pjax.loadUrl('/'+posts[Math.floor(Math.random()*posts.length)]) : window.open('/'+posts[Math.floor(Math.random()*posts.length)], "_self"); };</script>`

  return data

})
```

在主题配置文件引入 `_config.stellar.yml`，inject的 head里添加

```
- <script src="/js/random.js"></script> # 随机文章
```

然后在需要调用的位置执行 `toRandomPost()` 函数即可。比如任意 dom 添加 `onclick="toRandomPost()"`



好吧，我知道你肯定没听懂

反正我当时看完是一脸懵圈 不过没关系，我最后还是琢磨明白啦，下面就有填写示例，接着看就好![blobcat:ablobcatattentionreverse](Hexo-Stellar主题装修二/ablobcatattentionreverse-1749438579839-1054.png)

添加一个按钮:

随机阅读一篇文章



代码：`<button onclick="toRandomPost()">随机阅读一篇文章</button>`

或者添加一个链接: [随机阅读一篇文章](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#)

代码：`<a href="#" onclick="toRandomPost(); return false;">随机阅读一篇文章</a>`

在下一节还有应用示例，请往下看——

## 超链接样式调整[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#超链接样式调整)

文章内链接：加粗并下移下划线。显示效果：



![超链接样式](Hexo-Stellar主题装修二/CleanShot 2024-04-17 at 22.18.12@2x-1749438579839-1056.webp)超链接样式



在自定义 css 文件里加入：

```
/* 文章内链接 */

li:not([class]) a:not([class]),

p:not([class]) a:not([class]),

table a:not([class]) {

  /*color: var(--theme-link);*/

  padding-bottom: 3px; /* 增加底部padding */

  padding-right: 1px;

  margin-right: 2px;

  background: linear-gradient(0, var(--theme-link), var(--theme-link)) no-repeat center bottom / 100% 2px;

}
```

测试链接：[关于](https://www.flyalready.com/about/)

新样式！为链接使用荧光笔下划线效果，这个和上面的样式二选一就好。显示效果：

![img](Hexo-Stellar主题装修二/CleanShot 2024-04-20 at 19.32.04@2x-1749438579839-1058.webp)

```
/* 文章内链接：为链接使用荧光笔下划线效果 */

li:not([class]) a:not([class]),

p:not([class]) a:not([class]),

table a:not([class]) {

  padding-bottom: 0.1rem;

  background: linear-gradient(0, var(--theme-link-opa), var(--theme-link-opa)) no-repeat center bottom / 100% 40%;

}
```

## 选中文本：使用超链接高亮的背景色[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#选中文本使用超链接高亮的背景色)

在自定义 css 文件里加入：

```
/* 选中文本：使用超链接高亮的背景色 */

::selection {

  background: var(--theme-link-opa);

}
```

## Twikoo 评论样式优化[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#twikoo-评论样式优化)



Title

样式优化需要改主题文件，但下面的**给评论输入框加入提示**是纯 css 实现的不需要改

显示效果：

![img](Hexo-Stellar主题装修二/CleanShot 2024-04-17 at 02.13.48@2x-1749438579839-1060.webp)

只截了部分，整体效果可在评论区查看。代码全部抄自星日语大佬的[这条 commit](https://github.com/xaoxuu/hexo-theme-stellar/commit/3666dbd50a0ddf46a87635f51c9a08829ba41f9b#diff-b3cf1ac0cff1eefdab68933ee5005affaeb5b0781c8d63116a2ddecaf5ab00a1)。评论区表情显示优化可参考[这条 commit](https://github.com/xaoxuu/hexo-theme-stellar/commit/4567ceb8b883e142c3c4e7d84699b80676679e0c)。

### 给评论输入框加入提示[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#给评论输入框加入提示)

显示效果：

![img](Hexo-Stellar主题装修二/CleanShot 2024-04-17 at 02.17.46@2x-1749438579839-1062.webp)

原始代码忘记在哪里抄的了，我就修改了最后 3 行……在自定义 css 文件中加入以下内容：

```
/* 设置文字内容 :nth-child(1)的作用是选择第几个 */.el-input.el-input--small.el-input-group.el-input-group--prepend:nth-child(1):before {    content: '输入QQ号会自动获取昵称和头像🐧';}
.el-input.el-input--small.el-input-group.el-input-group--prepend:nth-child(2):before {    content: '收到回复将会发送到您的邮箱📧';}
.el-input.el-input--small.el-input-group.el-input-group--prepend:nth-child(3):before {    content: '填写后可以点击昵称访问您的网站🔗';}
/* 当用户点击输入框时显示 */.el-input.el-input--small.el-input-group.el-input-group--prepend:focus-within::before,.el-input.el-input--small.el-input-group.el-input-group--prepend:focus-within::after {    display: block;}
/* 主内容区 */.el-input.el-input--small.el-input-group.el-input-group--prepend::before {    /* 先隐藏起来 */    display: none;    /* 绝对定位 */    position: absolute;    /* 向上移动60像素 */    top: -60px;    /* 文字强制不换行，防止left:50%导致的文字换行 */    white-space: nowrap;    /* 圆角 */    border-radius: 10px;    /* 距离左边50% */    left: 50%;    /* 然后再向左边挪动自身的一半，即可实现居中 */    transform: translate(-50%);    /* 填充 */    padding: 14px 18px;    background: #444;    color: #fff;}
/* 小角标 */.el-input.el-input--small.el-input-group.el-input-group--prepend::after {    display: none;    content: '';    position: absolute;    /* 内容大小（宽高）为0且边框大小不为0的情况下，每一条边（4个边）都是一个三角形，组成一个正方形。    我们先将所有边框透明，再给其中的一条边添加颜色就可以实现小三角图标 */    border: 12px solid transparent;    border-top-color: #444;    left: 50%;    transform: translate(-50%, -48px);}
.el-input.el-input--small.el-input-group.el-input-group--prepend::before, .el-input.el-input--small.el-input-group.el-input-group--prepend::after {    z-index: 9999; /* 提高层级，确保内容显示在最前 */}
```

## Stellar & Twikoo 表情包补全计划[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#stellar--twikoo-表情包补全计划)

### blobcat[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#blobcat)

这个系列表情真的不要太可爱，一眼爱上

![blobcat:ablobcatattentionreverse](Hexo-Stellar主题装修二/ablobcatattentionreverse-1749438579839-1054.png) ![blobcat:ablobcatwave](Hexo-Stellar主题装修二/ablobcatwave-1749438579839-1064.png) ![blobcat:blobcatpresentred](Hexo-Stellar主题装修二/blobcatpresentred-1749438579839-1066.png)![blobcat:ablobcatknitsweats](Hexo-Stellar主题装修二/ablobcatknitsweats-1749438579839-1052.png)

光在博客正文里用怎么够，当然还要在评论区里也安排上![blobcat:ablobcatrainbow](Hexo-Stellar主题装修二/ablobcatrainbow-1749438579839-1068.png)

blobcat 表情主要来自[星日语](https://weekdaycare.cn/posts/emoji-blob/)佬。本人在学会自定义后收集癖大发，一口气制作了几个系列的表情，往现有的 blobcat里也加了几个比较好看的 ![blobcat:A_BlobCat_Code](Hexo-Stellar主题装修二/A_BlobCat_Code-1749438579839-1070.png)

Stellar 引入：`blobcatplus:https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/{name}.png`

Twikoo 使用链接：

```
https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/blobcatplus.json
```

| 表情                                                         | 索引                     | 表情                                                         | 索引                | 表情                                                         | 索引               |
| ------------------------------------------------------------ | ------------------------ | ------------------------------------------------------------ | ------------------- | ------------------------------------------------------------ | ------------------ |
| ![blobcat:ablobcatheart](Hexo-Stellar主题装修二/ablobcatheart-1749438579839-1042.png) | ablobcatheart            | ![blobcat:ablobcatheartbroken](Hexo-Stellar主题装修二/ablobcatheartbroken-1749438579839-1072.png) | ablobcatheartbroken | ![blobcat:blobcatheart](Hexo-Stellar主题装修二/blobcatheart-1749438579839-1074.png) | blobcatheart       |
| ![blobcat:blobcatheartpride](Hexo-Stellar主题装修二/blobcatheartpride-1749438579839-1076.png) | blobcatheartpride        | ![blobcat:blobcatlove](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatlove.png) | blobcatlove         | ![blobcat:blobcatkissheart](Hexo-Stellar主题装修二/blobcatkissheart-1749438579839-1080.png) | blobcatkissheart   |
| ![blobcat:blobcatsnuggle](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatsnuggle.png) | blobcatsnuggle           | ![blobcat:comfyuee](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/comfyuee.png) | comfyuee            | ![blobcat:comfyslep](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/comfyslep.png) | comfyslep          |
| ![blobcat:blobcatcomfysweat](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatcomfysweat.png) | blobcatcomfysweat        | ![blobcat:blobcatcomftears](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatcomftears.png) | blobcatcomftears    | ![blobcat:blobcatfacepalm](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatfacepalm.png) | blobcatfacepalm    |
| ![blobcat:blobcat0_0](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcat0_0.png) | blobcat0_0               | ![blobcat:blobcatangry](Hexo-Stellar主题装修二/blobcatangry-1749438579840-1096.png) | blobcatangry        | ![blobcat:blobbanhammerr](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobbanhammerr.png) | blobbanhammerr     |
| ![blobcat:blobcatt](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatt.png) | blobcatt                 | ![blobcat:blobcatblush](Hexo-Stellar主题装修二/blobcatblush-1749438579840-1102.png) | blobcatblush        | ![blobcat:blobcatcoffee](Hexo-Stellar主题装修二/blobcatcoffee-1749438579840-1104.png) | blobcatcoffee      |
| ![blobcat:blobcatcry](Hexo-Stellar主题装修二/blobcatcry-1749438579840-1106.png) | blobcatcry               | ![blobcat:blobcatdead](Hexo-Stellar主题装修二/blobcatdead-1749438579840-1108.png) | blobcatdead         | ![blobcat:blobcatdied](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatdied.png) | blobcatdied        |
| ![blobcat:blobcatdisturbed](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatdisturbed.png) | blobcatdisturbed         | ![blobcat:blobcatfearful](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatfearful.png) | blobcatfearful      | ![blobcat:blobcatfingerguns](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatfingerguns.png) | blobcatfingerguns  |
| ![blobcat:blobcatflip](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatflip.png) | blobcatflip              | ![blobcat:blobcatflower](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatflower.png) | blobcatflower       | ![blobcat:blobcatgay](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatgay.png) | blobcatgay         |
| ![blobcat:blobcatgooglycry](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatgooglycry.png) | blobcatgooglycry         | ![blobcat:blobcatneutral](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatneutral.png) | blobcatneutral      | ![blobcat:blobcatopenmouth](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatopenmouth.png) | blobcatopenmouth   |
| ![blobcat:blobcatsadreach](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatsadreach.png) | blobcatsadreach          | ![blobcat:blobcatscared](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatscared.png) | blobcatscared       | ![blobcat:blobcatnomblobcat](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatnomblobcat.png) | blobcatnomblobcat  |
| ![blobcat:blobcatpresentred](Hexo-Stellar主题装修二/blobcatpresentred-1749438579839-1066.png) | blobcatpresentred        | ![blobcat:blobcatread](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatread.png) | blobcatread         | ![blobcat:blobcatsipsweat](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatsipsweat.png) | blobcatsipsweat    |
| ![blobcat:blobcatsnapped](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatsnapped.png) | blobcatsnapped           | ![blobcat:blobcatthink](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatthink.png) | blobcatthink        | ![blobcat:blobcattriumph](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcattriumph.png) | blobcattriumph     |
| ![blobcat:blobcatumm](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatumm.png) | blobcatumm               | ![blobcat:blobcatverified](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatverified.png) | blobcatverified     | ![blobcat:blobcatbox](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatbox.png) | blobcatbox         |
| ![blobcat:blobcatcaged](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatcaged.png) | blobcatcaged             | ![blobcat:blobcatgooglytrash](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatgooglytrash.png) | blobcatgooglytrash  | ![blobcat:blobcatheadphones](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatheadphones.png) | blobcatheadphones  |
| ![blobcat:blobcathighfive](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcathighfive.png) | blobcathighfive          | ![blobcat:blobcatmelt](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatmelt.png) | blobcatmelt         | ![blobcat:blobcatmeltthumb](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatmeltthumb.png) | blobcatmeltthumb   |
| ![blobcat:blobcatnotlikethis](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatnotlikethis.png) | blobcatnotlikethis       | ![blobcat:blobcatsaitama](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatsaitama.png) | blobcatsaitama      | ![blobcat:blobcatyandere](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatyandere.png) | blobcatyandere     |
| ![blobcat:blobcatpeek2](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatpeek2.png) | blobcatpeek2             | ![blobcat:blobcatpeekaboo](Hexo-Stellar主题装修二/blobcatpeekaboo-1749438579838-1032.png) | blobcatpeekaboo     | ![blobcat:blobcatphoto](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatphoto.png) | blobcatphoto       |
| ![blobcat:ablobcatattentionreverse](Hexo-Stellar主题装修二/ablobcatattentionreverse-1749438579839-1054.png) | ablobcatattentionreverse | ![blobcat:ablobcatreachrev](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/ablobcatreachrev.png) | ablobcatreachrev    | ![blobcat:ablobcatwave](Hexo-Stellar主题装修二/ablobcatwave-1749438579839-1064.png) | ablobcatwave       |
| ![blobcat:blobcatalt](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatalt.png) | blobcatalt               | ![blobcat:blobcatpolice](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatpolice.png) | blobcatpolice       | ![blobcat:blobcatshocked](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcatshocked.png) | blobcatshocked     |
| ![blobcat:ablobcatrainbow](Hexo-Stellar主题装修二/ablobcatrainbow-1749438579839-1068.png) | ablobcatrainbow          |                                                              |                     |                                                              |                    |
| ![blobcat:A_BlobCat_REEEE](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/A_BlobCat_REEEE.png) | A_BlobCat_REEEE          | ![blobcat:A_BlobCat_Code](Hexo-Stellar主题装修二/A_BlobCat_Code-1749438579839-1070.png) | A_BlobCat_Code      | ![blobcat:ablobcatknitsweats](Hexo-Stellar主题装修二/ablobcatknitsweats-1749438579839-1052.png) | ablobcatknitsweats |
| ![blobcat:A_BlobCat_Nervous](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/A_BlobCat_Nervous.png) | A_BlobCat_Nervous        | ![blobcat:blobcat-aww](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/blobcat-aww.png) | blobcat-aww         | ![blobcat:ablobcatcry](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/ablobcatcry.png) | ablobcatcry        |
| ![blobcat:ablobcatdead](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/Blob/ablobcatdead.png) | ablobcatdead             |                                                              |                     |                                                              |                    |

### azuki[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#azuki)

![azuki:038](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/038.png)![azuki:039](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/039.png)![azuki:039](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/039.png)![azuki:039](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/039.png)![azuki:039](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/039.png)![azuki:040](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/040.png)

Stellar 引入：`azuki: https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/{name}.png`

Twikoo 使用链接：

```
https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/xiaodouni.json
```

| 表情                                                         | 索引 | 表情                                                         | 索引 | 表情                                                         | 索引 |
| ------------------------------------------------------------ | ---- | ------------------------------------------------------------ | ---- | ------------------------------------------------------------ | ---- |
| ![azuki:001](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/001.png) | 001  | ![azuki:015](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/015.png) | 015  | ![azuki:029](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/029.png) | 029  |
| ![azuki:002](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/002.png) | 002  | ![azuki:016](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/016.png) | 016  | ![azuki:030](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/030.png) | 030  |
| ![azuki:003](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/003.png) | 003  | ![azuki:017](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/017.png) | 017  | ![azuki:031](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/031.png) | 031  |
| ![azuki:004](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/004.png) | 004  | ![azuki:018](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/018.png) | 018  | ![azuki:032](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/032.png) | 032  |
| ![azuki:005](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/005.png) | 005  | ![azuki:019](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/019.png) | 019  | ![azuki:033](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/033.png) | 033  |
| ![azuki:006](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/006.png) | 006  | ![azuki:020](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/020.png) | 020  | ![azuki:034](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/034.png) | 034  |
| ![azuki:007](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/007.png) | 007  | ![azuki:021](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/021.png) | 021  | ![azuki:035](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/035.png) | 035  |
| ![azuki:008](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/008.png) | 008  | ![azuki:022](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/022.png) | 022  | ![azuki:036](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/036.png) | 036  |
| ![azuki:009](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/009.png) | 009  | ![azuki:023](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/023.png) | 023  | ![azuki:037](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/037.png) | 037  |
| ![azuki:010](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/010.png) | 010  | ![azuki:024](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/024.png) | 024  | ![azuki:038](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/038.png) | 038  |
| ![azuki:011](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/011.png) | 011  | ![azuki:025](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/025.png) | 025  | ![azuki:039](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/039.png) | 039  |
| ![azuki:012](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/012.png) | 012  | ![azuki:026](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/026.png) | 026  | ![azuki:040](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/040.png) | 040  |
| ![azuki:013](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/013.png) | 013  | ![azuki:027](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/027.png) | 027  |                                                              |      |
| ![azuki:014](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/014.png) | 014  | ![azuki:028](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/028.png) | 028  |                                                              |      |

### neko[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#neko)

![neko:038](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/038.png)![neko:039](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/039.png)![neko:040](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/040.png)

Stellar 引入：`neko: https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/{name}.png`

Twikoo 使用链接：

```
https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/neko.json
```

| 表情                                                         | 索引 | 表情                                                         | 索引 | 表情                                                         | 索引 |
| ------------------------------------------------------------ | ---- | ------------------------------------------------------------ | ---- | ------------------------------------------------------------ | ---- |
| ![neko:001](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/001.png) | 001  | ![neko:015](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/015.png) | 015  | ![neko:028](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/028.png) | 028  |
| ![neko:002](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/002.png) | 002  | ![neko:016](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/016.png) | 016  | ![neko:029](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/029.png) | 029  |
| ![neko:003](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/003.png) | 003  | ![neko:017](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/017.png) | 017  | ![neko:030](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/030.png) | 030  |
| ![neko:004](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/004.png) | 004  | ![neko:018](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/018.png) | 018  | ![neko:031](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/031.png) | 031  |
| ![neko:005](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/005.png) | 005  | ![neko:019](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/019.png) | 019  | ![neko:032](Hexo-Stellar主题装修二/032-1749438579782-1007.png) | 032  |
| ![neko:006](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/006.png) | 006  | ![neko:020](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/020.png) | 020  | ![neko:033](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/033.png) | 033  |
| ![neko:007](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/007.png) | 007  | ![neko:021](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/021.png) | 021  | ![neko:034](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/034.png) | 034  |
| ![neko:008](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/008.png) | 008  | ![neko:022](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/022.png) | 022  | ![neko:035](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/035.png) | 035  |
| ![neko:009](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/009.png) | 009  | ![neko:023](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/023.png) | 023  | ![neko:036](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/036.png) | 036  |
| ![neko:010](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/010.png) | 010  | ![neko:024](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/024.png) | 024  | ![neko:037](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/037.png) | 037  |
| ![neko:011](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/011.png) | 011  | ![neko:025](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/025.png) | 025  | ![neko:038](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/038.png) | 038  |
| ![neko:012](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/012.png) | 012  | ![neko:026](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/026.png) | 026  | ![neko:039](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/039.png) | 039  |
| ![neko:013](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/013.png) | 013  | ![neko:027](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/027.png) | 027  |                                                              |      |
| ![neko:014](https://cdn.jsdelivr.net/gh/2x-ercha/twikoo-magic@master/image/Yurui-Neko/014.png) | 014  |                                                              |      |                                                              |      |

### dokomo[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#dokomo)

Stellar 引入: `dokomo: https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/{name}.png`

Twikoo 使用链接:

```
https://raw.githubusercontent.com/infinitesum/Twikoo-emoji/main/dokomo/dokomo.json
```

| 表情                                                         | 索引      | 表情                                                         | 索引      | 表情                                                         | 索引      |
| ------------------------------------------------------------ | --------- | ------------------------------------------------------------ | --------- | ------------------------------------------------------------ | --------- |
| ![dokomo:dokomo-1](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-1.png) | dokomo-1  | ![dokomo:dokomo-18](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-18.png) | dokomo-18 | ![dokomo:dokomo-35](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-35.png) | dokomo-35 |
| ![dokomo:dokomo-2](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-2.png) | dokomo-2  | ![dokomo:dokomo-19](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-19.png) | dokomo-19 | ![dokomo:dokomo-36](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-36.png) | dokomo-36 |
| ![dokomo:dokomo-3](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-3.png) | dokomo-3  | ![dokomo:dokomo-20](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-20.png) | dokomo-20 | ![dokomo:dokomo-37](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-37.png) | dokomo-37 |
| ![dokomo:dokomo-4](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-4.png) | dokomo-4  | ![dokomo:dokomo-21](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-21.png) | dokomo-21 | ![dokomo:dokomo-38](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-38.png) | dokomo-38 |
| ![dokomo:dokomo-5](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-5.png) | dokomo-5  | ![dokomo:dokomo-22](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-22.png) | dokomo-22 | ![dokomo:dokomo-39](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-39.png) | dokomo-39 |
| ![dokomo:dokomo-6](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-6.png) | dokomo-6  | ![dokomo:dokomo-23](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-23.png) | dokomo-23 | ![dokomo:dokomo-40](Hexo-Stellar主题装修二/dokomo-40-1749438579846-1346.png) | dokomo-40 |
| ![dokomo:dokomo-7](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-7.png) | dokomo-7  | ![dokomo:dokomo-24](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-24.png) | dokomo-24 | ![dokomo:dokomo-41](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-41.png) | dokomo-41 |
| ![dokomo:dokomo-8](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-8.png) | dokomo-8  | ![dokomo:dokomo-25](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-25.png) | dokomo-25 | ![dokomo:dokomo-42](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-42.png) | dokomo-42 |
| ![dokomo:dokomo-9](Hexo-Stellar主题装修二/dokomo-9-1749438579846-1360.png) | dokomo-9  | ![dokomo:dokomo-26](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-26.png) | dokomo-26 | ![dokomo:dokomo-43](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-43.png) | dokomo-43 |
| ![dokomo:dokomo-10](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-10.png) | dokomo-10 | ![dokomo:dokomo-27](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-27.png) | dokomo-27 | ![dokomo:dokomo-44](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-44.png) | dokomo-44 |
| ![dokomo:dokomo-11](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-11.png) | dokomo-11 | ![dokomo:dokomo-28](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-28.png) | dokomo-28 | ![dokomo:dokomo-45](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-45.png) | dokomo-45 |
| ![dokomo:dokomo-12](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-12.png) | dokomo-12 | ![dokomo:dokomo-29](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-29.png) | dokomo-29 | ![dokomo:dokomo-46](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-46.png) | dokomo-46 |
| ![dokomo:dokomo-13](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-13.png) | dokomo-13 | ![dokomo:dokomo-30](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-30.png) | dokomo-30 | ![dokomo:dokomo-47](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-47.png) | dokomo-47 |
| ![dokomo:dokomo-14](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-14.png) | dokomo-14 | ![dokomo:dokomo-31](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-31.png) | dokomo-31 | ![dokomo:dokomo-48](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-48.png) | dokomo-48 |
| ![dokomo:dokomo-15](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-15.png) | dokomo-15 | ![dokomo:dokomo-32](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-32.png) | dokomo-32 | ![dokomo:dokomo-49](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-49.png) | dokomo-49 |
| ![dokomo:dokomo-16](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-16.png) | dokomo-16 | ![dokomo:dokomo-33](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-33.png) | dokomo-33 |                                                              |           |
| ![dokomo:dokomo-17](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-17.png) | dokomo-17 | ![dokomo:dokomo-34](https://cdn.jsdelivr.net/gh/infinitesum/Twikoo-emoji@master/dokomo/dokomo-34.png) | dokomo-34 |                                                              |           |

## 总字数统计：“发表了x篇文章，共计x字”[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#总字数统计发表了x篇文章共计x字)

需要修改主题文件 ![azuki:038](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/038.png)


```
// 3.left.top: 面包屑导航

  el += `<div class="flex-row" id="breadcrumb">`

    // 首页

    el += `<a class="cap breadcrumb" href="${url_for(config.root)}">${__("btn.home")}</a>`

    if (theme.wiki.tree[page.wiki]) {

      el += partial('breadcrumb/wiki')

    } else if (page.layout == 'post') {

      el += partial('breadcrumb/blog')

    } else {

      el += partial('breadcrumb/page')

    }

  // end 3.left.top

  el += `</div>`
```

并在后面添加：

```
 // 在这里添加标签代码

  if (page.layout == "post" && page.tags && page.tags.length > 0) {

    el += '<div id="tag">'; // 将标签容器的创建移动到条件内部

    el += ' <span>&nbsp标签：</span>';

    el += list_categories(page.tags, {

      class: "cap breadcrumb",

      show_count: false,

      separator: '&nbsp; ',

      style: "none"

    });

    el += '&nbsp</div>';

  }
```

## toc 字体大小调整[#](https://www.flyalready.com/site/hexo-stellar-主题装修笔记/#toc-字体大小调整)

需要修改主题文件 ![azuki:038](https://cdn.jsdelivr.net/gh/Saidosi/azuki-emoji-for-waline@1.0/azukisan/038.png)

就是把文章目录字体调小了一点点。

在`themes/stellar/source/css/_layout/widgets/toc.styl` 文件中，找到

```
// 各级缩进样式

.widget-wrapper.toc .toc

  .toc-item

    font-weight: 500

    --fsp: $fsp1

  .toc-item .toc-item

    font-weight: 400

    --fsp: $fsp2
```

把`--fsp: $fsp1`一行注释掉：

```
// 各级缩进样式

.widget-wrapper.toc .toc

  .toc-item

    font-weight: 500

    /*--fsp: $fsp1*/

  .toc-item .toc-item

    font-weight: 400

    --fsp: $fsp2
```
