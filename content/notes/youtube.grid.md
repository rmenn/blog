+++
date = '2025-11-14T19:53:36+05:30'
draft = false
title = 'Youtube Grid - Ublock Origin'
+++

adding this to the ublock origin custom filters lets you control the number of videos which show on the youtube grid, they have have been working on making it worse as time goes by to use on the browser


```
youtube.com##ytd-rich-grid-renderer, html:style(--ytd-rich-grid-items-per-row: 6 !important;)
```
