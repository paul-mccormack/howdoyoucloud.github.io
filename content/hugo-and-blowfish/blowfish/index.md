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

## Inserting images into pages

You can use standard markdown syntax to insert images into your pages.  For example:

```md
![Manchester](the-famous-manchester.jpg)
```

This will load the image file `the-famous-manchester.jpg` from the same folder as your markdown file.

![Manchester](the-famous-manchester.jpg)

> [!NOTE]
> The image file will only be available to this page as it is in the same folder as the markdown file.  If you want to use the same image on multiple pages then > it is better to store it in the `assets` folder and reference it from there.

You can also shrink the rendered image by adding custom attributes to the rendered HTML.  For example:

```md
![Shrunk Manchester](the-famous-manchester.jpg)
{style="width:50%;"}
```

![Shrunk Manchester](the-famous-manchester.jpg)
{style="width:50%;"}

You can also use the [figure](https://blowfish.page/docs/shortcodes/#figure) shortcode.

It works with animated images also!

![The Rocinante](the_roci.gif)

