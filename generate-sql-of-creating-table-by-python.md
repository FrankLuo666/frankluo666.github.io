---
title: "用python代码生成用于建表的sql语句"
date: 2023-03-26T17:57:48+09:00
draft: false
categories:
- IT
tags:
- python
- sql
- tool
thumbnailImage: "https://d1u9biwaxjngwg.cloudfront.net/highlighted-code-showcase/peak-140.jpg"
thumbnailImagePosition: left
autoThumbnailImage: yes
metaAlignment: center
summary: "失去人性，失去很多；失去兽性，失去一切。"
---



---

**需求：写一段python代码用来生产一个创建表的sql语句，其中传入的参数有：表名，字段名，字段的类型，字段的长度（如果是varchar类型），是否非空，是否是主键。**

可以创建一个名为`create_table_sql.py`的文件，代码示例：

```python
def generate_create_table_sql(table_name, columns):
    # table_name: str, the name of the table
    # columns: list of tuples, each tuple contains (column_name, data_type, length, not_null, primary_key)
    columns_sql = []
    for column in columns:
        column_name = column[0]
        data_type = column[1]
        length = column[2]
        not_null = "NOT NULL" if column[3] else ""
        primary_key = "PRIMARY KEY" if column[4] else ""
        if data_type == "varchar":
            column_sql = f"{column_name} {data_type}({length}) {not_null} {primary_key}"
        else:
            column_sql = f"{column_name} {data_type} {not_null} {primary_key}"
        columns_sql.append(column_sql)
    columns_str = ",\n".join(columns_sql)
    return f"CREATE TABLE {table_name} (\n{columns_str}\n);"

```

在需要使用它的地方导入它，如下所示：

```python
from create_table_sql import generate_create_table_sql

table_name = "my_table"
columns = [
    ("id", "int", None, True, True),
    ("name", "varchar", 50, True, False),
    ("age", "int", None, False, False),
    ("address", "varchar", 100, False, False),
]

create_table_sql = generate_create_table_sql(table_name, columns)
print(create_table_sql)

```

输出：

```sql
CREATE TABLE my_table (
id int NOT NULL PRIMARY KEY,
name varchar(50) NOT NULL,
age int,
address varchar(100)
);

```



---

**假设要求适用于PostgreSQL数据库，并且建表的时候要检查一下这个表是否存在，如果不存在才建表，可以使用以下代码：**

```python
import psycopg2  # 需要安装 psycopg2 包来连接 PostgreSQL 数据库


def generate_create_table_sql(table_name, columns):
    conn = psycopg2.connect(
        dbname="your_database_name",
        user="your_database_user",
        password="your_database_password",
        host="your_database_host",
        port="your_database_port"
    )
    cur = conn.cursor()

    # Check if table exists
    cur.execute(f"SELECT EXISTS (SELECT FROM information_schema.tables WHERE table_name = '{table_name}');")
    table_exists = cur.fetchone()[0]
    if table_exists:
        print(f"Table '{table_name}' already exists. Skipping table creation.")
        return

    # Create table
    columns_sql = []
    for column in columns:
        column_name = column[0]
        data_type = column[1]
        length = column[2]
        not_null = "NOT NULL" if column[3] else ""
        primary_key = "PRIMARY KEY" if column[4] else ""
        if data_type == "varchar":
            column_sql = f"{column_name} {data_type}({length}) {not_null} {primary_key}"
        else:
            column_sql = f"{column_name} {data_type} {not_null} {primary_key}"
        columns_sql.append(column_sql)
    columns_str = ",\n".join(columns_sql)
    create_table_sql = f"CREATE TABLE {table_name} (\n{columns_str}\n);"
    cur.execute(create_table_sql)
    conn.commit()

    print(f"Table '{table_name}' created successfully.")
    cur.close()
    conn.close()

```



