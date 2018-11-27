---
layout: post
title:  "启动mysql报错：The server quit without updating PID file *****"
date:   2017-05-30 23:00:13 +0000
categories: fermin update
---

启动mysql时报如下错误

```
> mysql.server start
Starting MySQL
. ERROR! The server quit without updating PID file (/usr/local/var/mysql/fyflzjzMBP.lan.pid).
```

这是由于mysql非正常关闭导致.pid文件没有清除
只需进入[/usr/local/var/mysql/]内将[fyflzjzMBP.*.pid]格式的pid文件删除即可

删除后再次重新启动mysql

```
> mysql.server start
Starting MySQL
. SUCCESS!
```

启动成功

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
