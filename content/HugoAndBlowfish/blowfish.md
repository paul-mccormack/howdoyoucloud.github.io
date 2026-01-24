---
title: "Blowfish Info, tips and tricks"
date: 2026-01-23
showDate: false
showReadingTime: false
showWordCount: false
draft: false
excludeFromSearch: true
---

## Exclude/Include sections in recent list on the homepage

This is set in the `config/_default/params.toml` file using the `mainSections` parameter.  You can include one or more sections here to have their most recent posts appear on the homepage Recent list. For example:

```toml
mainSections = ["posts", "projects"]
```

Blowfish expects this to be an array so if you only want section then still use the array format:

```toml
mainSections = ["posts"]
```