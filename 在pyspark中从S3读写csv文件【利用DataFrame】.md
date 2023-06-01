---
title: "在pyspark中从S3读写csv文件【利用DataFrame】"
date: 2023-04-19T11:52:36+09:00
draft: false
categories:
- IT
tags:
- python
- pyspark
- DataFrame
- AWS
---



以下是使用PySpark从AWS S3读取CSV文件，将其转换为DataFrame，提取一个字段的数据，然后将其写入S3的另一个路径的示例代码：

```python
from pyspark.sql import SparkSession

# 创建SparkSession
spark = SparkSession.builder.appName("ReadCSVFromS3").getOrCreate()

# 读取CSV文件并转换为DataFrame
df = spark.read.format("csv") \
            .option("header", "true") \
            .option("inferSchema", "true") \
            .load("s3://your-bucket-name/path-to-csv-file.csv")

# 提取一个字段的数据
column_data = df.select("column_name").rdd.flatMap(lambda x: x).collect()

# 创建包含提取的数据的DataFrame
new_df = spark.createDataFrame(column_data, "string").toDF("column_name")

# 将新的DataFrame写入S3的另一个路径
new_df.write.format("csv") \
            .option("header", "true") \
            .mode("overwrite") \
            .save("s3://your-bucket-name/new-path/")

```

在上面的代码示例中，我们没有指定新CSV文件的文件名。默认情况下，使用Spark写出的文件名是由Spark根据文件系统的默认算法生成的，通常是以部分哈希值和写入日期时间戳为基础。不过，您可以通过向`save()`方法传递文件名或路径来指定文件名。例如，以下代码演示了如何将新的DataFrame保存到名为"new_file.csv"的文件中：

```python
new_df.write.format("csv") \
            .option("header", "true") \
            .mode("overwrite") \
            .save("s3://your-bucket-name/new-path/new_file.csv")
```

这样就会在"s3://your-bucket-name/new-path/"路径下创建一个名为"new_file.csv"的新文件，并将新的DataFrame中的数据写入其中。如果您想要使用其他命名规则，可以在文件名中包含时间戳、分区键、文件类型等等，根据您的需要自定义。



---

如果您有多个CSV文件的路径，可以将这些路径存储在一个列表中，并使用`foreach()`循环遍历列表，读取每个CSV文件，提取所需的列，并将提取的数据合并到一个DataFrame中，最后将该DataFrame写入新的CSV文件中。下面是一个示例代码：

```python
from pyspark.sql import SparkSession

# 创建SparkSession
spark = SparkSession.builder.appName("ReadMultipleCSVsFromS3").getOrCreate()

# 要读取的CSV文件路径列表
csv_paths = ["s3://your-bucket-name/path-to-csv-file1.csv", 
             "s3://your-bucket-name/path-to-csv-file2.csv",
             "s3://your-bucket-name/path-to-csv-file3.csv"]

# 初始化空的DataFrame
combined_df = spark.createDataFrame([], "string").toDF("column_name")

# 循环遍历CSV文件路径列表
for csv_path in csv_paths:
    # 读取CSV文件并转换为DataFrame
    df = spark.read.format("csv") \
                .option("header", "true") \
                .option("inferSchema", "true") \
                .load(csv_path)

    # 提取"column_name"列的数据并合并到一个DataFrame中
    column_data = df.select("column_name").rdd.flatMap(lambda x: x).collect()
    new_df = spark.createDataFrame(column_data, "string").toDF("column_name")
    combined_df = combined_df.union(new_df)

# 将合并后的DataFrame写入新的CSV文件中
combined_df.write.format("csv") \
                .option("header", "true") \
                .mode("overwrite") \
                .save("s3://your-bucket-name/new-path/")
```

在上面的代码示例中，我们首先将要读取的CSV文件路径存储在一个列表中。然后，我们初始化一个空的DataFrame`combined_df`，用于存储提取的数据。接下来，我们使用`foreach()`循环遍历CSV文件路径列表，并读取每个CSV文件。对于每个CSV文件，我们提取所需的"column_name"列，并将其转换为一个新的DataFrame`new_df`，然后将其合并到`combined_df`中。最后，我们将合并后的DataFrame`combined_df`写入新的CSV文件中。



---

####  .option("inferSchema", "true") 

`inferSchema`是一个选项，用于自动推断DataFrame中每列的数据类型。如果将`inferSchema`设置为`true`，则Spark将尝试推断每列的数据类型，并将其设置为相应的数据类型。例如，如果某一列中的所有值都是整数，则Spark将该列的数据类型设置为整数类型。如果某一列中的值有整数和浮点数，则Spark将该列的数据类型设置为浮点数类型。

如果`inferSchema`设置为`false`，则Spark将默认将所有列的数据类型设置为字符串类型。

```python
df = spark.read.format("csv") \
            .option("header", "true") \
            .option("inferSchema", "false") \
            .load("s3://your-bucket-name/path-to-csv-file.csv")
```



需要注意的是，自动推断数据类型的过程可能会比较耗时，特别是在读取大型数据集时。如果您的数据集已经包含了列的数据类型信息，那么建议手动指定列的数据类型，以提高性能。例如，可以使用`df.withColumn("column_name", df["column_name"].cast("integer"))`的方式将某一列的数据类型设置为整数类型。



---

#### 如果您的数据集包含列的数据类型信息，可以手动指定列的数据类型，以提高性能。例如：

```python
from pyspark.sql.types import StructType, StructField, IntegerType, StringType

# 定义模式
schema = StructType([
    StructField("column_name1", StringType(), True),
    StructField("column_name2", IntegerType(), True),
    StructField("column_name3", StringType(), True)
])

# 读取CSV文件并应用模式
df = spark.read.format("csv") \
            .option("header", "true") \
            .schema(schema) \
            .load("s3://your-bucket-name/path-to-csv-file.csv")
```

上面的代码示例中，我们使用`StructType`定义了一个模式`schema`，并指定了每列的数据类型。然后，我们使用`.schema(schema)`选项将该模式应用到CSV文件中，以手动指定每列的数据类型。这样可以避免Spark进行数据类型推断，从而提高性能。
