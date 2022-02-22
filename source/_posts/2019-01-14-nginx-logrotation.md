---
title: nginx-logrotation-日志切割
date: 2019-01-14 23:36:14
tags:
---

# nginx日志

在 nginx 中，主要有 access 日志、error 日志 、 rewrite 日志。前两种由ngx_http_log_module模块予以支持，rewrite日志则由ngx_http_rewrite_module模块提供，这两个模块默认都已包含且启用。

- 