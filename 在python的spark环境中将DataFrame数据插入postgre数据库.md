---
title: "在python的spark环境中将DataFrame数据插入PostgreSQL数据库"
date: 2023-03-31T15:48:13+09:00
draft: false
categories:
- IT
tags:
- python
- pyspark
- DataFrame
- PostgreSQL
thumbnailImage: "https://d1u9biwaxjngwg.cloudfront.net/cover-image-showcase/city-750.jpg"
thumbnailImagePosition: left
autoThumbnailImage: yes
metaAlignment: center
summary: ""
---



### 创建一个dataframe

用pyspark创建一个dataframe，包含一个user表的信息，有id，name，gender，birthday等字段。

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import StructType, StructField, IntegerType, StringType, DateType
from pyspark.sql.functions import to_date

# 创建SparkSession
spark = SparkSession.builder.appName("Create Dataframe").getOrCreate()

# 定义用户信息列表
users = [
    {'id': 1, 'name': '张三', 'gender': '男', 'birthday': '19900101'},
    {'id': 2, 'name': '李四', 'gender': '女', 'birthday': '19921231'},
    {'id': 3, 'name': '王五', 'gender': '男', 'birthday': '19870615'}
]

# 定义dataframe的schema
schema = StructType([
    StructField("id", IntegerType(), True),
    StructField("name", StringType(), True),
    StructField("gender", StringType(), True),
    StructField("birthday_str", StringType(), True)
])

# 将用户信息列表转换为dataframe
df = spark.createDataFrame(users, schema=schema)

# 将birthday_str列转换为日期类型
df = df.withColumn('birthday', to_date(df['birthday_str'], 'yyyyMMdd'))

# 删除birthday_str列
df = df.drop('birthday_str')

# 打印dataframe
df.show()

```

以上示例代码中，我们首先创建了一个SparkSession，然后定义了包含三个用户信息的列表。接着，我们定义了dataframe的schema，其中包含三个字段，分别为id（整型）、name（字符串型）、gender（字符串型）和birthday（日期型）。最后，我们使用createDataFrame函数将用户信息列表转换为dataframe，并指定了schema。最终，我们调用show函数打印dataframe的内容，以验证数据是否正确地转换为了dataframe格式。

- StructField

  StructField是pyspark中用来定义dataframe中每个列的结构的类，其构造函数的参数包含三个部分，分别是：列名、数据类型和是否可空。

  需要注意的是，如果某列的数据类型为非空类型（如IntegerType），而其实际值为空，则会抛出异常。因此，在定义StructField时，需要根据实际情况来决定是否将某列的值设置为可空。

- 日期类型

  在以上示例代码中，我们首先添加了一个新的字段`birthday_str`，用于保存输入的日期字符串。然后，我们在schema中添加了一个新的StructField，用于表示`birthday_str`字段。接着，我们使用`createDataFrame`函数将用户信息列表转换为dataframe。

  在转换完成后，我们使用`withColumn`函数和`to_date`函数将`birthday_str`列转换为日期类型，并指定了日期格式为`'yyyyMMdd'`。最后，我们使用`drop`函数删除了`birthday_str`列，只保留了转换后的日期类型的`birthday`列。

  输出的dataframe中，`birthday`列的数据类型为`DateType`，格式为`yyyy-MM-dd`。

  

### 将dataframe数据插入到PostgreSQL数据库

- 按照每10条的方式提交一次。如果某一行的数据报错了，打印出行号。

如果你想将一个dataframe中的数据插入到PostgreSQL数据库中，并且想按照每10条数据一批的方式提交到数据库，可以使用`foreachPartition`函数将每个分区的数据分别插入到数据库中，并在插入每10条数据后提交一次事务。如果某一行的数据插入到数据库中报错了，打印出该行数据的行号，可以在`write_to_postgresql`函数中添加异常处理代码，并在异常处理代码中打印出行号。

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import StructType, StructField, IntegerType, StringType
import psycopg2

# 创建SparkSession
spark = SparkSession.builder.appName("Insert Data to PostgreSQL").getOrCreate()

# 定义用户信息列表
users = [
    {'id': 1, 'name': '张三', 'gender': '男', 'birthday': '1990-01-01'},
    {'id': 2, 'name': '李四', 'gender': '女', 'birthday': '1995-03-15'},
    {'id': 3, 'name': '王五', 'gender': '男', 'birthday': '1985-12-21'},
    # 其他17条数据
]

# 定义dataframe的schema
schema = StructType([
    StructField("id", IntegerType(), True),
    StructField("name", StringType(), True),
    StructField("gender", StringType(), True),
    StructField("birthday", StringType(), True)
])

# 将用户信息列表转换为dataframe
df = spark.createDataFrame(users, schema=schema)

# 将dataframe中的数据插入到PostgreSQL数据库中，并在插入过程中处理异常
def write_to_postgresql(rows):
    # 连接数据库
    conn = psycopg2.connect(
        host='localhost',
        port='5432',
        dbname='test_db',
        user='test_user',
        password='test_password'
    )
    cur = conn.cursor()
    try:
        # 插入数据
        for row in rows:
            try:
                cur.execute(
                    "INSERT INTO users(id, name, gender, birthday) VALUES (%s, %s, %s, %s)",
                    (row.id, row.name, row.gender, row.birthday)
                )
            except:
                print(f"Error inserting row {row.id}")
                raise
            # 每插入10条数据提交一次事务
            if cur.rowcount % 10 == 0:
                conn.commit()
        # 提交剩余的数据
        conn.commit()
    except:
        conn.rollback()
        raise
    finally:
        # 关闭数据库连接
        cur.close()
        conn.close()

# 将dataframe中的数据按照每个分区分别插入到数据库中
df.foreachPartition(write_to_postgresql)

```

在以上示例代码中，我们首先创建了一个dataframe，其中包含了20条用户信息数据。然后，我们定义了一个`write_to_postgresql`函数，用于将分区中的数据插入到PostgreSQL数据库中，并在插入每10条数据后提交一次事务。

我们使用`foreachPartition`函数将dataframe中的数据按照分区进行处理，并将每个分区的数据分别传递给`write_to_postgresql`函数进行处理。在`write_to_postgresql`函数中，我们首先连接到PostgreSQL数据库，然后逐行将数据插入到数据库中，每插入10条数据提交一次事务。

在`write_to_postgresql`函数中添加了异常处理代码，并在异常处理代码中打印出行号。如果某一行的数据插入到数据库中报错了，就会触发异常处理代码，打印出该行数据的行号。注意，在异常处理代码中我们要重新`raise`异常，以便将异常传递给Spark框架，以便在任务失败时能够及时停止任务。

最后，我们关闭数据库连接。

注意，在将dataframe中的数据插入到PostgreSQL数据库中时，我们需要根据实际情况设置数据库连接信息。另外，如果需要在代码中使用psycopg2库来连接PostgreSQL数据库，需要先使用pip安装该库。

- 如何raise异常

  在Python中，可以使用`raise`关键字来主动抛出异常。`raise`后面可以跟一个异常对象或异常类型，表示抛出该类型的异常。例如，下面的代码示例抛出一个`ValueError`类型的异常：

  ```python
  x = 10
  if x > 5:
      raise ValueError("x should be less than or equal to 5")
  ```

  在这个例子中，如果`x`的值大于5，就会抛出一个`ValueError`类型的异常，异常的信息是`"x should be less than or equal to 5"`。

  在实际应用中，可以在发生某些错误或异常情况时，使用`raise`关键字来抛出自定义的异常，以便让程序的其他部分能够捕获并处理这些异常。例如，在上一个问题的代码中，我们在异常处理代码中使用了`raise`关键字，以便将异常传递给Spark框架，以便在任务失败时能够及时停止任务。



### 使用Spark的DataFrame API将数据写入PostgreSQL数据库中

```python
# 将DataFrame写入PostgreSQL数据库中
dataframe.write \
    .mode('overwrite') \  # 覆盖模式，如果表已经存在，则删除并重新创建
    .format("jdbc") \  # 指定使用JDBC连接器
    .option("url", "jdbc:postgresql://<HOST>:<PORT>/<DATABASE>") \  # 指定PostgreSQL数据库的URL
    .option("dbtable", "<TABLE_NAME>") \  # 指定表名
    .option("user", "<USERNAME>") \  # 指定数据库用户名
    .option("password", "<PASSWORD>") \  # 指定数据库密码
    .option("driver", "org.postgresql.Driver") \  # 指定JDBC驱动程序
    .option("batchsize", "10000") \  # 指定每批次写入的数据量
    .option("truncate", "true") \  # 如果表已经存在，则截断表
    .option("isolationLevel", "NONE") \  # 事务隔离级别，这里设置为NONE
    .option("rewriteBatchedStatements", "true") \  # 开启批量处理
    .save()

# 日文翻译：
# PostgreSQLデータベースにDataFrameを書き込む
dataframe.write
  .mode('overwrite') \ # オーバーライトモード。テーブルが既に存在する場合は削除して再作成する
  .format("jdbc") \ # JDBCコネクタを使用することを指定
  .option("url", "jdbc:postgresql://<HOST>:<PORT>/<DATABASE>") \ # PostgreSQLデータベースのURLを指定
  .option("dbtable", "<TABLE_NAME>") \ # テーブル名を指定
  .option("user", "<USERNAME>") \ # データベースのユーザー名を指定
  .option("password", "<PASSWORD>") \ # データベースのパスワードを指定
  .option("driver", "org.postgresql.Driver") \ # JDBCドライバを指定
  .option("batchsize", "10000") \ # 1バッチあたりの書き込みデータ量を指定
  .option("truncate", "true") \ # テーブルが既に存在する場合は切り捨てる
  .option("isolationLevel", "NONE") \ # トランザクションの分離レベル。ここではNONEに設定する
  .option("rewriteBatchedStatements", "true") \ # バッチ処理を有効にする
  .save()
```

PostgreSQL数据库的URL格式为：`jdbc:postgresql://<HOST>:<PORT>/<DATABASE>`，其中`<HOST>`是数据库服务器的主机名或IP地址，`<PORT>`是PostgreSQL数据库的端口号，默认为5432，`<DATABASE>`是要连接的数据库名称。此外，由于使用了PostgreSQL数据库，因此需要指定相应的JDBC驱动程序，即`org.postgresql.Driver`。

- 其中，`rewriteBatchedStatements`是一个可选的选项，它可以在批量插入数据时提高性能。如果不设置这个选项，默认情况下是关闭的，也就是说不开启批量处理。因此，如果您不需要开启批量处理，可以不用设置这个选项。但是，如果您需要在写入大量数据时提高性能，建议开启这个选项。

- `truncate`是一个可选的选项，它可以在覆盖模式（overwrite mode）下控制表的行为。如果将`truncate`设置为`true`，则在写入数据之前，表将被截断（即删除表中的所有数据），然后再将数据写入表中。如果将`truncate`设置为`false`，则在写入数据之前，如果表已经存在，则会抛出异常。

  在这个特定的代码中，如果表已经存在，将使用覆盖模式（overwrite mode）来写入数据，因此如果设置了`truncate`为`true`，则在写入数据之前，表将被截断（即删除表中的所有数据），然后再将数据写入表中。如果没有设置`truncate`，则默认情况下会删除整个表并重新创建一个新表，因为覆盖模式会覆盖整个表，而不仅仅是表中的数据。因此，通过设置`truncate`为`true`，可以避免删除并重新创建整个表的操作，从而提高性能。

- `isolationLevel`是一个可选的选项，用于指定事务的隔离级别。事务的隔离级别是数据库管理系统（DBMS）中用于控制并发访问的一种机制。在多个用户同时访问同一个数据库时，可能会出现并发访问的问题，如脏读、不可重复读、幻读等。隔离级别可以控制这些问题的出现。

  常见的事务隔离级别有四种：READ UNCOMMITTED、READ COMMITTED、REPEATABLE READ和SERIALIZABLE。不同的隔离级别有不同的特点和适用场景。例如，READ UNCOMMITTED隔离级别允许一个事务读取另一个事务尚未提交的数据，而SERIALIZABLE隔离级别则会阻止所有并发访问，使得事务串行执行。

  在这个特定的代码中，`isolationLevel`被设置为`NONE`，表示不需要进行事务隔离。这意味着该操作不需要满足任何隔离级别的要求，可以并发执行。由于该操作是对数据进行批量写入，而且没有需要满足的事务隔离级别的要求，因此在这种情况下，设置`isolationLevel`为`NONE`是合理的。

  常见的事务隔离级别包括：

  - READ UNCOMMITTED（读未提交）：在该级别下，事务可以读取到其他事务未提交的数据。这种隔离级别通常会导致脏读、不可重复读和幻读等问题。一般情况下不建议使用该级别。
  - READ COMMITTED（读已提交）：在该级别下，事务只能读取到已经提交的数据。这可以避免脏读问题，但是可能会遇到不可重复读和幻读问题。
  - REPEATABLE READ（可重复读）：在该级别下，事务在执行过程中多次读取同一个数据时，可以保证读取到的数据是一致的。这种级别下可能会遇到幻读问题。
  - SERIALIZABLE（可串行化）：在该级别下，事务会按顺序一个接一个地执行，这样可以避免脏读、不可重复读和幻读等问题，但是会牺牲并发性能。

  根据不同的业务需求和数据访问特点，可以选择不同的事务隔离级别。一般情况下，选择隔离级别越高的级别，事务并发性能就越低，但是数据的一致性和可靠性会更高。



