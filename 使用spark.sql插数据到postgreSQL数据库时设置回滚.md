---
title: "使用spark.sql插数据到postgreSQL数据库时设置回滚"
date: 2023-04-10T09:45:20+09:00
draft: false
---



假设我们有两个DataFrame对象`df1`和`df2`，我们需要在一个事务中将它们插入到两个不同的表中，可以按照以下方式编写代码：

```python
# 启动事务
spark.sql("START TRANSACTION")

try:
    # 将df1插入到table1中
    df1.write.jdbc(url=url, table="table1", mode="append", properties=properties)
    
    # 将df2插入到table2中
    df2.write.jdbc(url=url, table="table2", mode="append", properties=properties)
    
    # 提交事务
    spark.sql("COMMIT")
except:
    # 回滚事务
    spark.sql("ROLLBACK")

```

在这个示例中，我们首先使用`START TRANSACTION`语句来启动一个事务，在`try`块中执行两个DataFrame的插入操作，最后使用`COMMIT`语句来提交事务。如果在执行操作时发生了异常，就会进入`except`块，并执行`ROLLBACK`语句来回滚事务。

需要注意的是，在使用DataFrame进行数据插入时，我们使用的是`write.jdbc()`方法，而不是使用SQL语句进行数据插入。但是，由于`write.jdbc()`方法内部会生成对应的SQL语句，因此上述方式同样适用于DataFrame的插入操作。




	
```
# 查询已存在的id列表
existing_ids = spark.read \
    .format("jdbc") \
    .option("url", jdbc_url) \
    .option("dbtable", table_name) \
    .option("user", properties["user"]) \
    .option("password", properties["password"]) \
    .load() \
    .select("id") \
    .collect()

# 将id列表转换为Python的集合
existing_ids = set([row.id for row in existing_ids])

# 创建一个要写入数据库的数据的列表
data_to_write = []

# 检查每行数据是否已存在
for row in df.collect():
    if row.id in existing_ids:
        # 如果id已经存在，则将数据更新
        data_to_write.append(row)
    else:
        # 如果id不存在，则将数据插入
        data_to_write.append(row)

# 将数据转换为dataframe
data_to_write = spark.createDataFrame(data_to_write, df.schema)

# 将数据写入数据库
data_to_write.write \
    .option("createTableColumnTypes", "id INT PRIMARY KEY, col1 TEXT, col2 TEXT") \
    .jdbc(jdbc_url, table_name, mode="overwrite", properties=properties)



```

