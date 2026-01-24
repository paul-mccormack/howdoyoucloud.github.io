---
title: "Blowfish Cheatsheet"
date: 2026-01-23
showDate: false
showReadingTime: false
showWordCount: false
draft: false
excludeFromSearch: true
---

![Blowfish Logo](blowfish.png)

This page is my personal cheatsheet for using the Blowfish theme for Hugo sites.  It is not intended to be a full guide to using Blowfish, just a quick reference for me to remember how to do things as I learn how to better use the theme.  If anyone else finds it useful then that's great! 😃

For full documentation on installing and using Blowfish please see the [official blowfish site](https://blowfish.page/)

## Exclude/Include sections in recent list on the homepage

This is set in the `config/_default/params.toml` file using the `mainSections` parameter.  You can include one or more sections here to have their most recent posts appear on the homepage Recent list. For example:

```toml
mainSections = ["posts", "projects"]
```

Blowfish expects this to be an array so if you only want section then still use the array format:

```toml
mainSections = ["posts"]
```