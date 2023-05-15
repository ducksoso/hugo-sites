---
title: "Basic Hugo"
date: 2023-05-12T14:18:37+08:00
draft: true
---





Here is content





## Front Matter



```yaml
---
title: "My First Post"
subtitle: ""
date: 2020-03-04T15:58:26+08:00
lastmod: 2020-03-04T15:58:26+08:00
draft: true
author: ""
authorLink: ""
description: ""
license: ""
images: []

tags: []
categories: []

featuredImage: ""
featuredImagePreview: ""

hiddenFromHomePage: false
hiddenFromSearch: false
twemoji: false
lightgallery: true
ruby: true
fraction: true
fontawesome: true
linkToMarkdown: true
rssFullText: false

toc:
  enable: true
  auto: true
code:
  copy: true
  maxShownLines: 50
math:
  enable: false
  # ...
mapbox:
  # ...
share:
  enable: true
  # ...
comment:
  enable: true
  # ...
library:
  css:
    # someCSS = "some.css"
    # located in "assets/"
    # Or
    # someCSS = "https://cdn.example.com/some.css"
  js:
    # someJS = "some.js"
    # located in "assets/"
    # Or
    # someJS = "https://cdn.example.com/some.js"
seo:
  images: []
  # ...
---
```

- **title**: the title for the content. --- 常用
- **subtitle**: the subtitle for the content.
- **date**: the datetime assigned to this page, which is usually fetched from the `date` field in front matter, but this behaviour is configurabl in the [site configuration](https://hugoloveit.com/theme-documentation-basics#site-configuration). --- 常用

  ```
  date: {{ .Date }}
  替代
  ```
  
- **lastmod**: the datetime at which the content was last modified.
- **draft**: if `true`, the content will not be rendered unless the `--buildDrafts`/`-D` flag is passed to the `hugo` command.  --- 常用
- **author**: the author for the content.  --- 常用
- **authorLink**: the link of the author.
- **description**: the description for the content.
- **license**: the special lisence for this content.
- **images**: page images for Open Graph and Twitter Cards.
- **tags**: the tags for the content.  --- 常用
- **categories**: the categories for the content.  --- 常用
- **featuredImage**: the featured image for the content.
- **featuredImagePreview**: the featured image for the content preview in the home page.
- **hiddenFromHomePage**: if `true`, the content will not be shown in the home page.
- **hiddenFromSearch**:  if `true`, the content will not be shown in the search results.
- **twemoji**: if `true`, the content will enable the twemoji.
- **lightgallery**: if `true`, images in the content will be shown as the gallery.
- **ruby**: if `true`, the content will enable the [ruby extended syntax](https://hugoloveit.com/theme-documentation-content/#ruby).
- **fraction**: if `true`, the content will enable the [fraction extended syntax](https://hugoloveit.com/theme-documentation-content/#fraction).
- **fontawesome**:  if `true`, the content will enable the [Font Awesome extended syntax](https://hugoloveit.com/theme-documentation-content/#fontawesome).
- **linkToMarkdown**: if `true`, the footer of the content will be shown the link to the orignal Markdown file.
- **rssFullText**: if `true`, the full text content will be shown in RSS.
- **toc**: the same as the `params.page.toc` part in the [site configuration](https://hugoloveit.com/theme-documentation-basics#site-configuration).
- **code**: the same as the `params.page.code` part in the [site configuration](https://hugoloveit.com/theme-documentation-basics#site-configuration).
- **math**: the same as the `params.page.math` part in the [site configuration](https://hugoloveit.com/theme-documentation-basics#site-configuration).
- **mapbox**:  the same as the `params.page.mapbox` part in the [site configuration](https://hugoloveit.com/theme-documentation-basics#site-configuration).
- **share**: the same as the `params.page.share` part in the [site configuration](https://hugoloveit.com/theme-documentation-basics#site-configuration).
- **comment**: the same as the `params.page.comment` part in the [site configuration](https://hugoloveit.com/theme-documentation-basics#site-configuration).
- **library**: the same as the `params.page.library` part in the [site configuration](https://hugoloveit.com/theme-documentation-basics#site-configuration).
- **seo**: the same as the `params.page.seo` part in the [site configuration](https://hugoloveit.com/theme-documentation-basics#site-configuration).







## summary



`<!--more-->`





参考：

https://hugoloveit.com/theme-documentation-content/
