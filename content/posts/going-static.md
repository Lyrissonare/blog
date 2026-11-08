---
title: 把服务器拿掉：一个博客的静态化记录
date: 2026-11-08 21:00
categories: [技术]
tags: [静态站点, GitHub Pages, 架构]
excerpt: 完整版 Cyanote 需要一台常驻的 Node 进程和一份 SQLite 文件。为了写几篇文章养一台服务器不划算，我把它搬进了静态文件，代价写在最后。
cover: "data:image/svg+xml;base64,PHN2ZyB4bWxucz0naHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmcnIHdpZHRoPScxMjAwJyBoZWlnaHQ9JzQ4MCcgdmlld0JveD0nMCAwIDEyMDAgNDgwJz48ZGVmcz48bGluZWFyR3JhZGllbnQgaWQ9J2cnIHgxPScwJyB5MT0nMCcgeDI9JzEnIHkyPScxJz48c3RvcCBvZmZzZXQ9JzAnIHN0b3AtY29sb3I9JyMwNTBiMWMnLz48c3RvcCBvZmZzZXQ9JzAuNTUnIHN0b3AtY29sb3I9JyMwYjJhNmInLz48c3RvcCBvZmZzZXQ9JzEnIHN0b3AtY29sb3I9JyMxZDRlZDgnLz48L2xpbmVhckdyYWRpZW50PjxyYWRpYWxHcmFkaWVudCBpZD0ncicgY3g9JzAuNzQnIGN5PScwLjMyJyByPScwLjYyJz48c3RvcCBvZmZzZXQ9JzAnIHN0b3AtY29sb3I9JyM2MGE1ZmEnIHN0b3Atb3BhY2l0eT0nMC4zNCcvPjxzdG9wIG9mZnNldD0nMScgc3RvcC1jb2xvcj0nIzYwYTVmYScgc3RvcC1vcGFjaXR5PScwJy8+PC9yYWRpYWxHcmFkaWVudD48L2RlZnM+PHJlY3Qgd2lkdGg9JzEyMDAnIGhlaWdodD0nNDgwJyBmaWxsPSd1cmwoI2cpJy8+PHJlY3Qgd2lkdGg9JzEyMDAnIGhlaWdodD0nNDgwJyBmaWxsPSd1cmwoI3IpJy8+PGcgZmlsbD0nbm9uZScgc3Ryb2tlPScjOTNjNWZkJyBzdHJva2Utb3BhY2l0eT0nMC4yNic+PGNpcmNsZSBjeD0nOTY1JyBjeT0nMjQyJyByPSc1OCcvPjxjaXJjbGUgY3g9Jzk2NScgY3k9JzI0Micgcj0nMTA4Jy8+PGNpcmNsZSBjeD0nOTY1JyBjeT0nMjQyJyByPScxNjQnLz48Y2lyY2xlIGN4PSc5NjUnIGN5PScyNDInIHI9JzIyNicvPjwvZz48ZyBmb250LWZhbWlseT0nc3lzdGVtLXVpLFNlZ29lIFVJLHNhbnMtc2VyaWYnPjx0ZXh0IHg9JzgwJyB5PScyMTInIGZvbnQtc2l6ZT0nNjQnIGZvbnQtd2VpZ2h0PSc3MDAnIGZpbGw9JyNmZmZmZmYnPuaKiuacjeWKoeWZqOaLv+aOiTwvdGV4dD48dGV4dCB4PSc4MicgeT0nMjcyJyBmb250LXNpemU9JzI1JyBmaWxsPScjYmZkYmZlJz7mioogRXhwcmVzcyArIFNRTGl0ZSDmjaLmiJDkuIDku70gSlNPTjwvdGV4dD48bGluZSB4MT0nODInIHkxPSczMjInIHgyPSczMDAnIHkyPSczMjInIHN0cm9rZT0nIzYwYTVmYScgc3Ryb2tlLW9wYWNpdHk9JzAuNTUnLz48dGV4dCB4PSc4MicgeT0nMzY2JyBmb250LXNpemU9JzE5JyBmaWxsPScjOTNjNWZkJyBsZXR0ZXItc3BhY2luZz0nNic+Q1lBTk9URSBMSVRFPC90ZXh0PjwvZz48L3N2Zz4="
---

# 把服务器拿掉

十月的某个晚上，我躺在床上想到一句话，希望它第二天出现在博客上。完整版 Cyanote 办不到，它要求我先打开电脑，起一个 Node 进程，进 `/admin`，把字敲进编辑框，点保存。

这台机器平时不声不响，摊开来看是这样：一个 Express 进程，一份 SQLite 文件，一个首次启动时自动生成的管理令牌，一套按文件头校验图片的上传接口，四条限流规则，一个按 IP 每小时计一次的浏览量。

这些东西没有一样是我想要的。我想要一个能写字的地方。

## 先写下要留下的

动手前列了两张单子。留下的有：首页、文章列表、分类、标签、归档、关于、暗色主题、代码高亮、`#/post/slug` 这样的地址。丢掉的有：在线编辑器、令牌鉴权、图片上传、限流、浏览量写回、分类与标签的在线改名。

分界线很好找。写作需要服务器，阅读不需要。所有会写入的动作都在 `/admin` 后面，公开站点从头到尾只读。

这条线一画，工程量就有了形状。前台 `public/js` 893 行，管理台 `admin/js` 953 行，服务端 `server` 1156 行。要搬走的是最前面那 893 行，要扔掉的是后面两千行。

## 切在接缝上

前端和数据之间只有一个入口。`js/api.js` 导出 `get()`，六个页面模块 import 它，标签页通过 `site.js` 的 `getMeta()` 落到同一个入口。没有一个模块知道数据是从 HTTP 来的，还是从别处来的。

完整版的 `get()` 打的是一串 REST 地址。公开页面用到的只读接口有五个：`/api/meta`、`/api/settings`、`/api/archive`、`/api/articles`、`/api/articles/:slug`。静态版把这五个地址的解释权搬进浏览器，路径和返回结构一字不改。

```js
export async function api(path, opts = {}) {
  if ((opts.method || 'GET').toUpperCase() !== 'GET') {
    throw new Error('Cyanote Lite 是纯静态站点：请在本地编辑 Markdown 后重新构建');
  }
  return handleGet(path);
}
```

写操作在静态站里没有正确的实现。唯一诚实的回应是报错，并且告诉调用者正确的事情在本地做。

索引会被反复读取。`getManifest()` 把 fetch 的 Promise 存在模块作用域里，一页里多个组件问它要数据，只发一次请求。请求失败时把缓存清空，下一次调用重新发，网络抖一下不会让整页卡在错误状态。

前端为这件事改了两处。`index.html` 里的 `/css/app.css` 改成 `css/app.css`，这样在 GitHub Pages 的项目子路径下也能取到资源。另一处是 `api.js`。除了 `home.js` 为首页那个溶解动画多接了一个模块，页面模块、`markdown.js`、`util.js`、`site.js` 都原样搬过来。

## 把查询搬进浏览器

服务端的 `listArticles` 按状态过滤，按分类和标签过滤，按关键词搜索，置顶优先再按时间排序，再切片分页。`store.js` 用 78 行把这些语义复刻了一遍。

```js
items = [...items].sort(
  (a, b) => (Number(b.pinned) - Number(a.pinned)) ||
            (order === 'asc' ? newestFirst(b, a) : newestFirst(a, b))
);
```

这几十行里有三个容易写错的点。置顶的比较要排在日期前面，否则置顶只在第一页有效。`pageSize` 要夹在 1 到 100 之间，和原来的后端一致，页面传 0 或者 500 都不会出错。归档接口返回的字段叫 `published_at`，蛇形命名和别处不一致，我留着它，因为归档页读的就是这个名字。

搜索是三处 `includes`：标题、正文、摘要，统一转小写。中文没有分词，两个字的词和一整句话走同一条路径，对上就算命中。文章少的时候够用，文章多的时候这是最先需要换掉的一块。

侧边栏的分类云和标签云来自 `/api/meta`，只统计已发布的文章，按数量倒序。数量一样时谁在前谁在后，由索引里的原始顺序决定，而那个顺序在建索引时按创建时间固定下来。改动排序规则，侧边栏的并列顺序就会跟着变，这种耦合不看代码是猜不到的。

草稿过滤了两遍。构建时草稿不进索引，浏览器里再按 `status` 过一遍。多这一遍不费什么，少这一遍会漏。

## 文章变回文件

数据库里一篇文章有十几个字段。搬进 Markdown 之后，开头那段 front-matter 承担同样的职责。

```markdown
---
title: 把服务器拿掉
date: 2026-11-08 21:00
categories: [技术]
tags: [静态站点, GitHub Pages, 架构]
excerpt: 一句话摘要，留空就截正文前 150 字
pinned: false
views: 0
status: published
---
```

slug 由文件名决定，`hello-cyanote.md` 就是 `#/post/hello-cyanote`。这个约定省掉一个字段，也让地址在文件系统里读得懂。

日期有个坑。front-matter 里写 `2026-11-08 21:00`，没有时区。JavaScript 默认按本地时区解析，北京和伦敦构建出来的 `publishedAt` 差八个小时，同一份源文件在两台机器上生成不同字节的 JSON。构建器把不带时区的日期一律按 UTC 解释，这个差异就消失了。

草稿不写进索引，文件仍然留在仓库里。公开仓库里的草稿是公开的，`status: draft` 只挡住列表页，挡不住 `git clone`。

front-matter 的解析器是手写的，160 行，支持标量、`[a, b]` 内联数组、短横线列表、引号、布尔和数字，没有引 YAML 库。发布器里有一份 C# 的同样实现，两边的语法必须对齐，否则同一个文件在网页推送和本地推送两条路上会被读成两种样子。

## 为什么不用现成的

Hexo、Hugo、Astro 都能把一堆 md 变成站点，用它们更省事。省掉的是构建器，代价是把这套页面按模板语法重写：样式怎么进、路由怎么配、主题切换和代码高亮怎么接，全部要重新做一遍。我想要的只是换掉数据源，页面留在原处。一百多行的构建器，比学一套模板语法快。

## 构建器

`build.mjs` 做的事很少。读 `content/posts` 下的 md，解析 front-matter，按创建时间排一次序，拼成一个 JSON，写回 `content/index.json`。

5 毫秒。四篇文章本来就该这么快，这个数字没什么可说的。花心思的地方在确定性。发布器推送文章时会顺带更新索引，CI 在每次推送后也会再构建一次。两边只要差一个字节，仓库里就多一个没有意义的提交。

所以键的顺序固定，数组排序稳定，日期统一 UTC，文件结尾留一个换行。写完这一篇，索引 18KB，gzip 之后 9KB，四篇全文都在里面。

目录里出现两个同名 slug 时，构建直接报错退出，不会悄悄覆盖其中一个。

## 迁移

数据库只读打开一次。`node:sqlite` 的 `DatabaseSync` 支持 `readOnly`，整个迁移过程里原目录不会被写坏。

文章按 id 顺序导出，`categories` 和 `tags` 从 JSON 字符串还原成数组，正文里的 `/uploads/xxx.png` 全部改写成 `content/images/xxx.png`，图片文件跟着拷过来。设置表整个导成 `settings.json`。

跑完之后，`data` 目录 4.5MB 的东西变成 `content` 目录 33KB 的文本。那个 4.5MB 里，数据库文件本身只有 44KB，其余是 WAL：几个月的写作让文字反复进出，旧页面都留在日志里。数据库没有删，留在原处当备份。

## 发布

写作回到文件。VS Code、Obsidian、记事本都行，改完 `git push`。

手机上想改一个错别字，GitHub 网页的编辑框也能提交。仓库里挂了一个 `rebuild-index` 工作流，只改 md 不碰索引时，它会把索引补齐。

发布器是另一条路。`publisher/` 下是一个 .NET 8 的 WPF 程序，选文件、补 front-matter、点推送，走 GitHub Contents API 一次提交两样东西：文章的 md 和更新后的索引。

部署分成两个仓库。`Cyanote_Lite` 放引擎，`blog` 放内容和页面。Pages 的工作流把两者检出到同一个工作区，把内容覆盖过去，跑一次构建，上传。

```yaml
- run: |
    rm -rf engine/content
    cp -R blog/content engine/content
    node engine/build.mjs
```

升级引擎和写文章互不打扰。改模板只动 `Cyanote_Lite`，推送之后在 `blog` 的 Actions 里手动跑一次部署。

## 代价

浏览量会冻结。静态页没有地方写回数字，迁移时把历史值抄进 front-matter，新文章永远是 0。要让计数重新动起来，得接一个第三方统计，外部依赖又回来了。

搜索是全量内存过滤。索引里存着每篇文章的全文，读者打开首页就把这 18KB 全部下载完。文章到几百篇时，文件会变成几百 KB，首屏要等它。

没有在线编辑器。改一个错别字也要走一次提交。

评论要挂外部服务，我没有做。分类改名靠全文查找替换，比点两下鼠标慢，也比点两下鼠标可靠。

这些是账单。静态化省下的是进程、数据库和运维。

也有几件事顺手变好了。备份是一个 `git clone`，换机器不用导数据库。攻击面从一个能写文件、能收上传的 Web 服务缩到一堆静态文件，没有令牌可泄漏，没有上传目录可以塞东西。每一篇文章的修改都留 diff，写坏了能回滚到某一段话，而不是从备份里翻出一个整库。

## 回到那个晚上

这篇文章现在就是 `content/posts` 下的一个 md 文件。写完，`node build.mjs`，`git push`，随后出现在 https://lyrissonare.github.io/blog/ 上。没有进程需要照看，也没有数据库需要备份。

前端比原版多了一个开屏动画，WebGL 的水波和鱼眼，一千多行。它和轻量没有关系，是我想看。

深夜想到一句话，掏出手机，在 GitHub 的编辑框里敲三行，提交。两分钟后它就在那里。
