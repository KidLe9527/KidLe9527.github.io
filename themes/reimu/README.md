<img src="https://cdn.jsdelivr.net/gh/D-Sketon/hugo-theme-reimu@main/images/screenshot.png"/>
<div align = center>
  <h1>hugo-theme-reimu</h1>
  <img alt="version" src="https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fgithub.com%2FD-Sketon%2Fhugo-theme-reimu%2Fraw%2Fmain%2Fpackage.json&query=%24.version&label=version">
  <img alt="GitHub License" src="https://img.shields.io/github/license/D-Sketon/hugo-theme-reimu">
  <img alt="GitHub Repo stars" src="https://img.shields.io/github/stars/D-Sketon/hugo-theme-reimu">
  <p align="center">
  💘 博麗 霊夢 💘
  </p>

[演示网站](https://d-sketon.github.io/hugo-theme-reimu) | [迁移指南](https://d-sketon.github.io/20241110/hexo-to-hugo-guide/) | [Change Log](https://github.com/D-Sketon/hugo-theme-reimu/blob/main/CHANGELOG.md)

简体中文 | [English](https://github.com/D-Sketon/hugo-theme-reimu/blob/main/README.en.md)

</div>

A Hakurei Reimu style Hugo theme. Migrated from [hexo-theme-reimu](https://github.com/D-Sketon/hexo-theme-reimu).

---

| framework                    | repository                                                         | version                                                                                                                                                                                     | stars                                                                                              |
| ---------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| [Hexo](https://hexo.io/)     | [hexo-theme-reimu](https://github.com/D-Sketon/hexo-theme-reimu)   | <img alt="version" src="https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fgithub.com%2FD-Sketon%2Fhexo-theme-reimu%2Fraw%2Fmain%2Fpackage.json&query=%24.version&label=version">  | <img alt="GitHub Repo stars" src="https://img.shields.io/github/stars/D-Sketon/hexo-theme-reimu">  |
| [Astro](https://astro.build) | [astro-theme-reimu](https://github.com/D-Sketon/astro-theme-reimu) | <img alt="version" src="https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fgithub.com%2FD-Sketon%2Fastro-theme-reimu%2Fraw%2Fmain%2Fpackage.json&query=%24.version&label=version"> | <img alt="GitHub Repo stars" src="https://img.shields.io/github/stars/D-Sketon/astro-theme-reimu"> |
| [Hugo](https://gohugo.io)    | [hugo-theme-reimu](https://github.com/D-Sketon/hugo-theme-reimu)   | <img alt="version" src="https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fgithub.com%2FD-Sketon%2Fhugo-theme-reimu%2Fraw%2Fmain%2Fpackage.json&query=%24.version&label=version">  | <img alt="GitHub Repo stars" src="https://img.shields.io/github/stars/D-Sketon/hugo-theme-reimu">  |

**项目刚刚开始，欢迎提出 Issue 和 PR！**

## 特性

### 基础功能

- ✨ 完整的博客功能
- 🔄 兼容 Hugo 0.124.0 及以上版本
- 📱 响应式布局
- 🌙 暗黑模式支持
- 🅰️ i18n 支持

### 代码与数学

- 🖥️ 代码高亮与复制
- ➗ KaTeX / MathJax3 数学公式支持
- 📊 Mermaid 流程图支持

### 搜索与评论

- 🔍 Algolia 搜索集成
- 💬 多评论系统支持：
  - Valine
  - Waline
  - Twikoo
  - Gitalk
  - Giscus

### 统计与分析

- 📊 文章阅读统计（Valine / Waline）
- 👥 访客统计（不蒜子）

### 媒体与交互功能

- 🎵 音乐播放器支持：
  - Aplayer
  - Meting
- 🖼️ 图片懒加载
- ⚡ 加载动画
- 🖱️ 鼠标特效：
  - 动画效果
  - 灵梦鼠标指针
- 👾 Live2D / Live2D-widgets 集成

### 导航与结构

- 📑 目录导航（TOC）
- 🔄 PJAX 支持
- 🔧 ServiceWorker 实现
- 📰 RSS 订阅

### 设计与自定义

- 🎨 图标支持：
  - Iconfont
  - FontAwesome
- 🔗 自定义短代码：
  - 内部链接
  - 外部链接
  - 友情链接
  - 热力图
  - 标签轮盘
- 🎨 动态适配主题色
- ©️ 文章版权声明
- 🌐 自定义 CDN 源 / 本地配置
- 📜 自定义字体
- 🎨 分享卡片功能

## 安装

> 纯小白可以直接使用 [hugo-reimu-template](https://github.com/D-Sketon/hugo-reimu-template)。只需 克隆仓库-修改配置 即可获得一个基本的博客！

### 方式 1：Hugo Module（推荐）

适用于熟悉 Go 生态的用户，支持版本管理和自动更新。

```bash
hugo new site quickstart
cd quickstart
hugo mod init github.com/<your-github-username>/quickstart  # 请替换 <your-github-username>
echo 'theme = ["github.com/D-Sketon/hugo-theme-reimu"]' >> hugo.toml
hugo server
```

### 方式 2：Git Submodule

适用于希望手动管理主题版本的用户。

```bash
hugo new site quickstart
cd quickstart
git init
git submodule add https://github.com/D-Sketon/hugo-theme-reimu.git themes/reimu
echo 'theme = "reimu"' >> hugo.toml
hugo server
```

选择适合你的安装方式，即可开始使用 **Hugo Theme Reimu**！

## 使用

<details>
<summary>创建配置</summary>

### 创建配置

#### 主题配置

在外层 `config` 文件夹下创建 `_default` 文件夹，然后将主题内的 `config/_default/params.yml` 复制到 `_default` 文件夹下，此文件作为主题配置文件，可在此文件中修改主题配置

#### 数据配置

将主题内的 `config/data/` 文件夹内的所有文件复制到外层 `data` 文件夹下，此文件夹内的文件用于配置主题内的数据：

- `covers.yml` 用于配置随机封面图片
- `friends.yml` 用于配置友链
- `vendor.yml` 用于配置第三方库的 CDN 源

#### 静态资源配置

主题的静态资源（favicon，头图等）位于 `static` 文件夹内，你可以在外层 `static` 文件夹下创建相应的文件夹，然后将主题内的文件复制到外层文件夹下，以覆盖主题内的默认文件

> 总之一句话，不建议直接修改主题内的文件，而是在外层创建相应的文件夹，然后将主题内的文件复制到外层文件夹下，以覆盖主题内的默认文件，这样便于主题升级

#### 语言配置

reimu 支持多语言。如果你想要使用中文，请在 `hugo.toml` 中添加如下配置

```toml
languageCode = 'zh-CN'
defaultContentLanguage = 'zh-CN'
[languages]
[languages.zh-CN]
languageName = '简体中文'
weight = 1
hasCJKLanguage = true
```

</details>
<details>

<summary>基本结构</summary>

### 基本结构

为了保证显示正确，请参考 `_example` 在 `content` 中建立 `archives` 和 `post` 文件夹 （里面的 `_index.md` 不可省略，且注意 `post` 的 `draft` 为 `true`）

#### archives

- `_index.md` 用于显示归档页面，不可省略

#### post

文章请在此目录下创建，注意 `draft` 为 `true` 的文章不会显示在首页

- `_index.md` 用于忽略 `post/index.html` 的生成，不可省略

#### about\.md

关于页面

#### friend\.md

友链页面

</details>
<details>
<summary>头像、封面、头图和图标</summary>

### 头像、封面、头图和favicon

#### 头像

头像应保存于 `static/avatar/avatar.webp`，可在 `params.yml` 中修改文件名

```yaml
avatar: "avatar.webp"
```

#### 封面

随机封面图片请参考主题内的 `data/covers.yml` 文件结构，在外层 `data` 文件夹下创建 `covers.yml` 文件，格式如下：

```yaml
- https://example.com/1.jpg
- https://example.com/2.jpg
```

封面显示逻辑如下

- 如果文章的 Front matter 中包含 cover 的 url，则该文章头图和首页缩略图均显示该 url

```yaml
---
title: Hello World
cover: https://example.com
---
```

- 如果文章的 Front matter 中包含 cover 为 `false`，则该文章不显示头图（首页上仍然是随机图片）

```yaml
---
title: Hello World
cover: false
---
```

- 如果文章的 Front matter 中包含 cover 为 `rgb(xxx,xxx,xxx)`，则该文章头图为对应的渐变纯色（首页上仍然是随机图片）

```yaml
---
title: Hello World
cover: rgb(255,117,117)
---
```

- 否则查找 `data` 文件夹中的 `covers.yml`，并从中随机挑选图片
- 若上述文件均不存在，则显示头图

#### 头图

头图保存于 `themes/hugo-theme-reimu/static/images/banner.webp`，可在 `params.yml` 中修改路径和名字

```yaml
banner: "images/banner.webp"
```

#### Favicon

favicon 保存于 `themes/hugo-theme-reimu/static/favicon.ico`，可自行覆盖替换

</details>
<details>
<summary>代码块</summary>

### 代码块

为保证代码块的正确显示，请保证 `hugo.toml` 中有如下配置

```toml
[markup.highlight]
guessSyntax = true
noClasses = false
```

代码块同时提供了代码粘贴功能，点击代码块右上角的复制按钮即可复制代码。在 `params.yml` 中可以对复制功能进行配置。  
`success` 为复制成功时的提示，`fail` 为复制失败时的提示。此外，可以配置版权声明，当复制的字符数大于 `count` 时会在复制的内容后面添加版权声明。

```yaml
clipboard:
  success:
    en: Copy successfully (*^▽^*)
    zh-CN: 复制成功 (*^▽^*)
    zh-TW: 複製成功 (*^▽^*)
    ja: コピー成功 (*^▽^*)
  fail:
    en: Copy failed (ﾟ⊿ﾟ)ﾂ
    zh-CN: 复制失败 (ﾟ⊿ﾟ)ﾂ
    zh-TW: 複製失敗 (ﾟ⊿ﾟ)ﾂ
    ja: コピー失敗 (ﾟ⊿ﾟ)ﾂ
  copyright:
    enable: false
    count: 50 # 大于多少字符添加版权声明
    license_type: by-nc-sa # https://creativecommons.org/licenses
```

v0.2.0 添加了配置用于控制代码块的默认展开状态，`expand` 可以设置为 `true`、`false` 或数字，数字表示当代码块的行数大于该数字时默认收缩。

```yaml
code_block:
  expand: true # true | false | number
```

</details>
<details>
<summary>站内评论</summary>

### 站内评论

> 站内评论可以使用 Front matter 中的 `comments` 独立控制每篇文章是否显示评论。  
> 当 `comments` 为 `false` 时不显示评论，`true` 或不填时根据 `params.yml` 的配置决定是否显示。

> 0.8.0+ 后支持多评论系统同时使用

全局评论系统配置：

```yaml
comment:
  title: # 评论框标题
    en: Leave a comment
    zh-CN: 说些什么吧！
    zh-TW: 說些什麼吧！
    ja: コメントを残す
  default: waline # 多评论下，默认使用的评论系统
```

若基于 [Valine](https://valine.js.org/)  
请参考其官方文档完成 `LeanCloud` 的配置，并在 `params.yml` 中将 `valine.enable` 改为 `true`，并填入自己的 `appId` 和 `appKey`

```yaml
valine:
  enable: true
  appId: "your appId"
  appKey: "your appKey"
```

若基于 [Waline](https://waline.js.org/)  
请参考其[官方文档](https://waline.js.org/guide/get-started/)完成 `LeanCloud` 的配置，并在 `params.yml` 中将 `waline.enable` 改为 `true`，并填入自己的 `serverURL`

```yaml
waline:
  enable: true
  serverURL: "your server url"
  locale: {} # https://waline.js.org/guide/features/i18n.html#%E8%87%AA%E5%AE%9A%E4%B9%89%E8%AF%AD%E8%A8%80
  emoji:
    - https://unpkg.com/@waline/emojis@1.2.0/weibo
    - https://unpkg.com/@waline/emojis@1.2.0/alus
    - https://unpkg.com/@waline/emojis@1.2.0/bilibili
    - https://unpkg.com/@waline/emojis@1.2.0/qq
    - https://unpkg.com/@waline/emojis@1.2.0/tieba
    - https://unpkg.com/@waline/emojis@1.2.0/tw-emoji
  meta:
    - nick
    - mail
    - link
  requiredMeta:
    - nick
    - mail
  wordLimit: 0
  pageSize: 10
  pageview: true
```

若基于 [twikoo](https://twikoo.js.org)  
请参考其[官方文档](https://twikoo.js.org/quick-start.html)完成 腾讯云 或 Vercel 部署，并在 `params.yml` 中将 `twikoo.enable` 改为 `true`，并填入自己的 `envId`

```yml
twikoo:
  enable: true
  envId: # 腾讯云环境填 envId；Vercel 环境填地址（https://xxx.vercel.app）
  region:
```

若基于 [giscus](https://giscus.app/zh-CN)  
请参考文档完成仓库的配置，并在 `params.yml` 中将 `giscus.enable` 改为 `true`，并填入对应的数据

```yml
giscus:
  enable: true
  repo: "your repo"
  repoId: "your repoId"
  category: "your category"
  categoryId: "your categoryId"
  mapping: mapping
  strict: 0
  reactionsEnabled: 1
  emitMetadata: 0
  inputPosition: bottom
```

若基于 [gitalk](https://gitalk.github.io/)  
请参考其[官方文档](https://github.com/gitalk/gitalk?tab=readme-ov-file#usage)完成仓库的配置，并在 `params.yml` 中将 `gitalk.enable` 改为 `true`，并填入对应的数据

```yml
gitalk:
  enable: true
  clientID: "your application client ID"
  clientSecret: "your application client secret"
  repo: "your repo"
  owner: "repo owner"
  admin: "repo owner and collaborators"
  md5: false # 是否使用 md5 加密路径
```

</details>

<details>
<summary>站内搜索</summary>

### 站内搜索

基于 [Algolia](https://www.algolia.com/)，请在外层 `hugo.toml` 中添加如下配置

```toml
[outputs]
home = ["Algolia", "HTML", "RSS"]

[outputFormats.Algolia]
baseName = "algolia"
isPlainText = true
mediaType = "application/json"
notAlternative = true
```

这样会在 `public` 文件夹下生成 `algolia.json` 文件，用于 Algolia 搜索。接着你可以使用诸如 `atomic-algolia` 等插件将其上传到 Algolia

同时，在 `params.yml` 中将 `algolia_search.enable` 改为 `true`，并填写相关信息（**注意！这里填写的是Search-Only Key，不允许填写Admin Key！！否则可能被攻击**）

```yaml
algolia_search:
  enable: true
```

</details>
<details>
<summary>数学公式</summary>

### 数学公式

请先在外层 `hugo.toml` 中添加如下配置

```toml
[markup.goldmark.extensions.passthrough]
enable = true
delimiters.block = [["\\[", "\\]"], ["$$", "$$"]]
delimiters.inline = [["\\(", "\\)"], ["$", "$"]]
```

并在需要使用数学公式的文章的 Front matter 中添加 `math` 为 `true`

```yaml
---
math: true
---
```

> 注意不要同时开启 KaTeX 和 MathJax3

#### KaTeX

若基于 [Katex](https://github.com/KaTeX/KaTeX)，请在 `params.yml` 中将 `math.katex.enable` 改为 `true`

```yaml
math:
  katex:
    enable: true
```

#### MathJax3

若基于 [MathJax3](https://www.mathjax.org/)，请在 `params.yml` 中将 `math.mathjax.enable` 改为 `true`，并可在 `options` 中添加配置（由于 Hugo 会自动将对象的key转变为全小写，所以配置需要放在一个数组中避免默认行为）

```yaml
math:
  mathjax:
    enable: true
    options: [{}]
```

</details>
<details>
<summary>Mermaid</summary>

### Mermaid

流程图基于 [Mermaid](https://mermaid.js.org/#/)，请在需要使用流程图的文章的 Front matter 中添加 `mermaid` 为 `true`

```yaml
---
mermaid: true
---
```

</details>
<details>
<summary>RSS</summary>

### RSS

RSS 可以在 `params.yml` 中配置

```yaml
rss:
  limit: 10 # 输出最近的文章数量，写 -1 输出全部
  showFullContent: false # 是否输出文章的完整内容
  showCopyright: false # 是否输出版权声明
```

</details>

<details>
<summary>Icon</summary>

### Icon

Icon 默认使用本项目提供的 iconfont

```yml
icon_font: 4552607_0khxww3tj3q9
```

如果想要继续使用 fontawesome 图标，请将 `icon_font` 设置为 `false`，此时会使用 `vendor.yml` 中对应的 fontawesome

```yml
fontawesome:
  high_priority:
    - src: webcache|@fortawesome/fontawesome-free@6.5.1/css/regular.min.css
      integrity: sha384-k5640LgghgAohDLPwSqVWa96yQwWouT6wsAL+J1g0CFJVITNKYkIh1XpPLYKQe7Y
    - src: webcache|@fortawesome/fontawesome-free@6.5.1/css/solid.min.css
      integrity: sha384-8yO/A/BtltnG0hDxdwmmkza8UAleyDoAD1FhXiH6rsOQQsCho1P6WZP9TpBBH3YP
  low_priority:
    - src: webcache|@fortawesome/fontawesome-free@6.5.1/css/brands.min.css
      integrity: sha384-/BRyRRN0wxxRgh/DAXU621go9pdoMHl6LFPiX5Pp8PZYZlKBQCDXj9X9DHx6LOud
    - src: webcache|@fortawesome/fontawesome-free@6.5.1/css/v5-font-face.min.css
      integrity: sha384-/mBKnLlGtog8q2qQrgugURRDV+iHWHAPvM5KulYXT1C2ErKOKkBI0vbff8ZPq7rL
    - src: webcache|@fortawesome/fontawesome-free@6.5.1/css/v4-font-face.min.css
      integrity: sha384-d2Yn1/9Iw78r3oqwk5B+EcpRcmepXR5LyhmRF2a+WoSe9mpRGvVk0ZviFwDGDOTO
```

</details>

<details>
<summary>扩展功能</summary>

### 扩展功能

#### 暗黑模式

默认为 `auto`，根据用户系统设置自动切换。可以设置为 `true` 或 `false` 改变默认状态

```yaml
dark_mode:
  # true 代表暗黑模式默认开启
  # false 代表暗黑模式默认关闭
  # auto 代表根据用户系统设置自动切换
  enable: auto # true | false | auto
```

#### Pace 进度条

默认开启

```yaml
pace:
  enable: true
```

#### Firework 鼠标特效

默认开启

```yaml
firework:
  enable: true
```

具体配置请查看 [mouse-firework](https://github.com/D-Sketon/mouse-firework)

#### PJAX

默认关闭

```yaml
pjax:
  enable: false
```

> PJAX 用于那些需要添加音乐播放器等需要 SPA 的用户。但其仍然属于实验性质，引入后可能会出现诸如**脚本无法执行**、**脚本重复执行**、**页面渲染混乱**等 BUG。请慎重考虑！

#### ServiceWorker

默认关闭

```yaml
service_worker:
  enable: false
```

#### Live2D

默认关闭

```yaml
live2d:
  enable: false
  position: left # left | right
```

#### Live2D Widgets

默认关闭

```yaml
live2d_widgets:
  enable: false
  position: left # left | right
```

#### Reimu 鼠标指针

默认开启

```yml
reimu_cursor: true
```

#### 响应式头图

默认关闭，打开后并提供对应尺寸的图片和媒体查询可以在一定程度上提高移动端的 LCP

```yml
banner_srcset:
enable: false
srcset:
  - src: "images/banner-600w.webp"
    media: "(max-width: 479px)"
  - src: "images/banner-800w.webp"
    media: "(max-width: 799px)"
  - src: "images/banner.webp"
    media: "(min-width: 800px)"
```

#### Quicklink

默认开启，打开后可以在用户停留在页面时预加载链接，提高用户体验

```yml
quicklink:
  enable: true
  timeout: 3000 # 预加载超时时间
  priority: true # 是否优先加载
  ignores: [] # 忽略的链接，仅支持字符串
```

#### 文章版权声明

默认关闭

```yml
article_copyright:
  enable: false # 是否展示版权卡片？
  content:
    author: # true | false 版权卡片展示作者？
    link: # true | false 版权卡片展示链接？
    title: # true | false 版权卡片展示标题？
    date: # true | false 版权卡片展示创建日期？
    updated: # true | false 版权卡片展示更新日期？
    license: # true | false 版权卡片展示协议？
    license_type: by-nc-sa # https://creativecommons.org/licenses
```

此外，也可以通过文章的 front-matter 控制，其优先级高于全局配置

```yaml
---
copyright: true # 是否展示版权卡片？
---
```

#### 文章过期提醒

默认关闭

```yml
outdate:
  enable: false
  daysAgo: 180 # 多少天前的文章算过期
  message:
    en: This article was last updated on {time}. Please note that the content may no longer be applicable.
    zh-CN: 本文最后更新于 {time}，请注意文中内容可能已不适用。
    zh-TW: 本文最後更新於 {time}，請注意文中內容可能已不適用。
    ja: この記事は最終更新日：{time}。記載内容が現在有効でない可能性がありますのでご注意ください。
```

#### 赞助

默认关闭

```yml
sponsor:
  enable: false # 是否展示赞助二维码？
  tip: # 赞助提示
    zh-CN: 请作者喝杯咖啡吧
    zh-TW: 請作者喝杯咖啡吧
    en: Buy me a coffee
    ja: コーヒーを買ってください
  icon:
    url: "../images/taichi.png" # 赞助图标，相对于 css/main.css 的路径，所以需要向上一级才能找到 images 文件夹
    rotate: true # 是否旋转图标
    mask: true # 是否将图片作为遮罩（即只显示 png 图片的轮廓）
  qr:
    - name: 支付宝 # 二维码名称
      src: "sponsor/alipay.jpg" # 示例二维码路径，位于 static/sponsor/aliapy.jpg
```

此外，也可以通过文章的 front-matter 控制，其优先级高于全局配置

```yaml
---
sponsor: true # 是否展示赞助二维码？
---
```

#### 音乐播放器（v0.4.0+）

> 使用前建议先打开 Pjax，否则会出现播放器自动暂停的问题

使用Aplayer + Meting（可选）默认关闭

##### 纯Aplayer

将 `player.aplayer.enable` 设置为 `true`，并在 `player.aplayer.options` 中参考 [Aplayer Docs](https://aplayer.js.org/#/home?id=options) 进行配置

```yml
player:
  aplayer:
    enable: true
    options:
      audio: [] # audio list
      fixed:
      autoplay:
      loop:
      order:
      preload:
      volume:
      mutex:
      listFolded:
      lrcType:
```

##### Aplayer + Meting

同时将 `player.aplayer.enable` 和 `player.meting.enable` 设置为 `true`，并在 `player.meting.options` 中参考 [Meting Docs](https://github.com/metowolf/MetingJS?tab=readme-ov-file#option) 进行配置，`player.aplayer.options` 为 Aplayer 配置

```yml
player:
  aplayer:
    enable: true
    options:
      audio: [] # this option will be overwritten by meting
      fixed:
      autoplay:
      loop:
      order:
      preload:
      volume:
      mutex:
      listFolded:
      lrcType:
  meting:
    enable: true
    meting_api: # custom api
    options:
      id:
      server:
      type:
      auto:
```

#### Pangu 自动分割

默认关闭，自动替你在文章中所有的中文字和半形的英文、数字、符号之间插入空白。

```yml
pangu:
  enable: false
```

#### 分享链接/卡片（v0.5.0+）

默认关闭，目前支持 `facebook`、`twitter`、`linkedin`、`reddit`、`weibo`、`qq`、`weixin`。

```yml
share:
  # - facebook
  # - twitter
  # - linkedin
  # - reddit
  # - weibo
  # - qq
  # - weixin
```

`weixin` 状态下会生成带有二维码的分享卡片，可保存到本地后分享到微信朋友圈（注意，当文章封面存在跨域问题时无法使用 html-to-image 正确生成含图片的卡片！）

#### 首页目录卡片（v0.6.0+）

默认关闭，打开后可以在首页展示目录卡片，用于代替 widget 中的目录

```yml
home_categories:
  enable: false # 是否展示首页目录卡片？
  content:
    - categories: # 目录名称，字符串类型
      cover: # 卡片封面，不填则使用随机封面
    - categories:
      cover:
```

</details>

<details>
<summary>内置卡片shortcode</summary>

### 内置卡片shortcode

#### friendLink 友链卡片

```yaml
{{< friendsLink >}}
```

无参数，直接读取 `data/friends.yml` 文件

#### postLinkCard 内链卡片

```yaml
{{<postLinkCard path="?" cover="?" escape="?" >}}
```

其中第一个参数为文章的 `path`；第二个参数（可选）为卡片展示的封面，如果设置为 `auto` 则自动使用博客的 `banner`；第三个参数（可选，`true | false`）表示文章标题是否被转义

#### externalLinkCard 外链卡片

```yaml
{{<externalLinkCard title="?" link="?" cover="?">}}
```

其中第一个参数为文章的标题；第二个参数为文章的外部链接，第三个参数（可选）为卡片展示的封面，如果设置为 `auto` 则自动使用缺省封面

#### heatMapCard 文章热力图 (v0.8.0+ 实验性功能)

```yaml
{{< heatMapCard levelStandard="?" >}}
```

其中第一个参数为热力图的等级标准（按照文章字数分级），默认为 `"1000,5000,10000"`

#### tagRoulette 标签轮盘 (v0.12.0+ 实验性功能)

```yaml
{{< tagRoulette tags="?" icon="?" >}}
```

tagRoulette 是一个互动元素，提供随机标签展示功能，点击按钮后会从预定义的标签池中随机抽取并展示一个标签。

- tags：可选参数，指定标签池，多个标签用英文逗号(,)分隔；未提供时默认使用几个示例标签，例如：tags="记忆衰退,表达欲丧失,更加怠惰,无感,好想睡觉"  
- icon：可选参数，自定义触发按钮的图标，默认使用：🕹️（游戏手柄emoji），可替换为任何emoji或文字，如 🎲、🎯、🔄 等
</details>

<details>
<summary>自定义主题</summary>

#### 动态适配主题色 (v0.8.0+ 实验性功能)

默认关闭，打开后会基于 Google's Material You 的设计规范根据文章头图的主色调动态生成主题色

```yml
material_theme:
  enable: false # true | false
```

> 注意：当开启该功能时，会在 banner 的 img 元素上添加 `crossorigin="anonymous"` 属性，以获取图片的主色调，所以请确保你的图片服务器支持跨域访问，或使用第三方图片代理。

#### 手动定制主题颜色

hugo-theme-reimu 主题支持通过 CSS 变量定制主题颜色，你可以通过修改 `:root` 伪类下的 CSS 变量来定制你的主题颜色。

~~变量文件位于 `assets/css/_variables.scss`，你可以在这个文件中找到所有的 CSS 变量，但其实只需要修改以下伪类下的变量即可~~

v0.9.0 对外暴露了 `internal_theme` 配置用于定制主题颜色 token

```yaml
internal_theme:
  light:
    --red-0: "#ff0000"
    --red-1: "#ff5252"
    --red-2: "#ff7c7c"
    --red-3: "#ffafaf"
    --red-4: "#ffd0d0"
    --red-5: "#ffecec"
    --red-5-5: "#fff3f3"
    --red-6: "#fff7f7"
    --color-red-6-shadow: "rgba(255, 78, 78, 0.6)"
    --color-red-3-shadow: "rgba(255, 78, 78, 0.3)"

    --highlight-nav: "#e6e6e6"
    --highlight-scrollbar: "#d6d6d6"
    --highlight-background: "#f7f7f7"
    --highlight-current-line: "#dadada"
    --highlight-selection: "#e9e9e9"
    --highlight-foreground: "#4d4d4d"
    --highlight-comment: "#7d7d7d"
    --highlight-red: "#c8362b"
    --highlight-orange: "#b66014"
    --highlight-yellow: "#cb911d"
    --highlight-green: "#2ea52e"
    --highlight-aqua: "#479d9d"
    --highlight-blue: "#1973b8"
    --highlight-purple: "#7135ac"
  dark:
    --red-4: "rgba(255, 208, 208, 0.5)"
    --red-5: "rgba(255,228,228,0.15)"
    --red-5-5: "rgba(255,236,236,0.05)"
    --red-6: "rgba(255, 243, 243, 0.2)"

    --highlight-nav: "#2e353f"
    --highlight-scrollbar: "#454d59"
    --highlight-background: "#22272e"
    --highlight-current-line: "#393939"
    --highlight-selection: "#515151"
    --highlight-foreground: "#cccccc"
    --highlight-comment: "#999999"
    --highlight-red: "#f47067"
    --highlight-orange: "#f69d50"
    --highlight-yellow: "#ffcc66"
    --highlight-green: "#99cc99"
    --highlight-aqua: "#66cccc"
    --highlight-blue: "#54b6ff"
    --highlight-purple: "#dcbdfb"
```

#### 自定义字体

可通过以下配置定义谷歌字体：

```yaml
# https://fonts.google.com/
font:
  enable: true # 是否启用谷歌字体
  article:
    - Mulish
    - Noto Serif SC
  code:
    # - Ubuntu Mono
    # - Source Code Pro
    # - JetBrains Mono
```

v0.2.0 添加了 `local_font` 配置用于定义本机字体，其优先级比谷歌字体低：

```yaml
local_font:
  article:
    - "-apple-system"
    - PingFang SC
    - Microsoft YaHei
    - sans-serif
  code:
    - Menlo
    - Monaco
    - Consolas
    - monospace
```

v0.9.0 添加了 `custom_font` 配置用于定义自定义字体，其优先级最高：

```yaml
custom_font:
  enable: true
  article:
    - css: https://fontsapi.zeoseven.com/292/main/result.css # 字体 css 文件
      name: LXGW WenKai # 字体名称
  code:
```

#### 定制图标

##### 头部 / 侧边栏图标

v0.1.0 的 `menu` 配置的结构发生了变化，允许用户自定义 icon。icon 为空时默认使用太极图标，你可以填写一个十六进制的数字来自定义 icon，同时支持 fontawesome 和 icon font。

v0.10.2 icon 支持图片路径，如 `/avatar/avatar.webp`。

```yaml
menu:
  - name: home
    url: /
    icon: # 不填默认使用太极图标
  - name: archives
    url: /archives
    icon: f0c1 # 你可以填写一个十六进制的数字来自定义 icon，支持 fontawesome 和 icon font
  - name: about
    url: /about
    icon:
  - name: friend
    url: /friend
    icon:
```

##### 底部 / 回到顶部 / 赞助图标

v0.1.0 的 `footer`、`top`、`sponsor` 配置均增加了 `icon` 配置用于自定义图标。

- `url` 为图标的路径，相对于 `css/main.css` 的路径，所以需要向上一级才能找到 images 文件夹。
- `rotate` 为是否旋转图标，默认为 `true`。
- `mask` 是否将图片作为遮罩（即只显示 png 图片的轮廓），默认为 `true`。

```yaml
footer:
  icon:
    url: "../images/taichi.png" # 相对于 css/main.css 的路径，所以需要向上一级才能找到 images 文件夹
    rotate: true
    mask: true

top:
  icon:
    url: "../images/taichi.png"
    rotate: true
    mask: true

sponsor:
  icon:
    url: "../images/taichi.png"
    rotate: true
    mask: true
```

##### 加载图标

v0.1.0 的 `preloader` 配置增加了 `icon` 配置用于自定义图标。icon 为空时默认使用内链的 svg（保证首屏加载速度），你可以填入一个链接来自定义加载图标。

不建议使用过大的图标，以免影响加载速度。

```yaml
preloader:
  enable: true
  text:
    zh-CN: 少女祈祷中...
    zh-TW: 少女祈禱中...
    en: Loading...
    ja: 少女祈祷中...
  icon: # 不填默认使用内链的svg（保证首屏加载速度），你可以填入一个链接来自定义加载图标，如 '/images/taichi.png'
  rotate: true
```

##### 锚点图标

v0.1.0 增加了 `anchor_icon` 配置用于自定义锚点图标，默认使用 `#` 图标，你可以填写一个十六进制的数字来自定义 icon，同时支持 fontawesome 和 icon font。

```yaml
anchor_icon: # 不填默认使用 # 图标
```

v0.11.0 `anchor_icon` 支持传递 `false` 以隐藏锚点图标

##### 鼠标图标（v0.5.0+）

v0.5.0 增加了 `reimu_cursor.cursor` 配置用于自定义鼠标图标，你可以填写一个相对于 `css/main.css` 的路径来自定义鼠标图标。

```yaml
reimu_cursor:
  enable: true
  cursor:
    default: ../images/cursor/reimu-cursor-default.png
    pointer: ../images/cursor/reimu-cursor-pointer.png
    text: ../images/cursor/reimu-cursor-text.png
```

</details>

<details>
<summary>Vendor</summary>

### Vendor

`vendor` 用于存放一些第三方资源，如 fontawesome、iconfont、katex、mathjax 等。

hugo-theme-reimu 的 `vendor` 结构非常灵活，其支持以下几种形式：

- `:cdn|:package@:version/:file`：使用 CDN 加速，如 `cdn_jsdelivr_gh|katex@0.13.11/dist/katex.min.css`，`:cdn`可在 `vendor` 中自行配置。目前自带以下 CDN 源：
  ```yaml
  cdn_jsdelivr_gh: https://cdn.jsdelivr.net/gh/ # 仅针对github加速
  cdn_jsdelivr_npm: https://cdn.jsdelivr.net/npm/ # 仅针对npm加速
  fastly_jsdelivr_gh: https://fastly.jsdelivr.net/gh/ # 仅针对github加速
  fastly_jsdelivr_npm: https://fastly.jsdelivr.net/npm/ # 仅针对npm加速
  unpkg: https://unpkg.com/ # 仅针对npm加速
  webcache: https://npm.webcache.cn/ # 仅针对npm加速
  local: /resources/ # 本地资源
  ```
  用户可根据网络状况自行切换 CDN 源。
- `https://:path` 开头：直接使用绝对链接，如 `https://cdn.jsdelivr.net/npm/katex@0.13.11/dist/katex.min.css`
- `:path` 开头：本地资源，你可以把资源放在 `static` 文件夹下，然后使用诸如 `katex.min.css` 的路径引用

此外，`vendor` 还支持 SRI 校验，你可以在 `vendor` 中使用 `SHA-384` 用于校验资源的完整性，如：

```yaml
js:
  clipboard: # 使用 SRI 校验
    src: webcache|clipboard@2.0.11/dist/clipboard.min.js
    integrity: sha384-J08i8An/QeARD9ExYpvphB8BsyOj3Gh2TSh1aLINKO3L0cMSH2dN3E22zFoXEi0Q
  lazysizes: webcache|lazysizes@5.3.2/lazysizes.min.js # 不使用 SRI 校验
```

以上两种形式均支持，建议对外部 CDN 资源使用 SRI 校验，以确保资源的完整性。

</details>

## 贡献者

[![](https://contributors-img.web.app/image?repo=D-Sketon/hugo-theme-reimu)](https://github.com/D-Sketon/hugo-theme-reimu/graphs/contributors)

## 赞助 💘

[爱发电-afdian](https://afdian.tv/a/dsketon)

## 许可

MIT
