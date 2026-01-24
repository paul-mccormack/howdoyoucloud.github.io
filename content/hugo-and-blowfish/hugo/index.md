---
title: "Hugo Cheatsheet"
date: 2026-01-23
showDate: false
showReadingTime: false
showWordCount: false
draft: false
excludeFromSearch: true
---

This page is my personal cheatsheet for using the Hugo.  It is not intended to be a full guide, just a quick reference for me to remember how to do things as I learn how to better use Hugo.  If anyone else finds it useful then that's great! 😃

For full documentation on installing and using Hugo please see the [official Hugo site](https://gohugo.io/)

## Managing Content

Hugo uses Markdown files for content. You can create new content using the Hugo CLI:

```bash
hugo new posts/my-first-post.md
```

This will create a new Markdown file in the `content/posts/` directory with some default front matter. You can then edit this file to add your content.  The default front matter can be configured in the `archetypes` directory.