---
title: "在python中将字符串转为日期型插入到postgreSQL数据库"
date: 2023-04-11T17:02:35+09:00
draft: false
---

可以使用Python中的`datetime`模块来将字符串转换为日期型，并使用`strftime()`函数将其格式化为适合PostgreSQL数据库的日期型格式。具体来说，你可以使用以下代码将字符串转换为日期型并将其格式化为`yyyy-MM-dd`的日期型：

```python
from datetime import datetime

date_str = "2023-02-01"
date_obj = datetime.strptime(date_str, "%Y-%m-%d")
date_formatted = date_obj.strftime("%Y-%m-%d")

```

在这个代码中，`datetime.strptime()`函数将字符串解析为`datetime`对象，然后`strftime()`函数将`datetime`对象格式化为`yyyy-MM-dd`的日期型字符串。如果你要将日期型数据存储到PostgreSQL数据库中，你可以将`date_formatted`变量的值插入到数据库中。
