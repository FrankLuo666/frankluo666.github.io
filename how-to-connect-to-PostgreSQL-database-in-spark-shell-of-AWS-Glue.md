---
title: "在AWS Glue的Spark脚本中编写连接和断开连接PostgreSQL数据库的方法"
date: 2023-03-25T15:46:00+09:00
draft: false
categories:
- IT
tags:
- AWS
- pyspark
- postgreSQL
thumbnailImage: "https://d1u9biwaxjngwg.cloudfront.net/tag-plugins-showcase/car-6-140.jpg"
thumbnailImagePosition: left
autoThumbnailImage: yes
metaAlignment: center
summary: "弱小和无知不是生存的障碍，傲慢才是。"
---



可以使用JDBC连接器来实现，示例代码：

```python
# 导入所需的类
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext,SparkConf
from awsglue.context import GlueContext
from awsglue.dynamicframe import DynamicFrame
from pyspark.sql.function import *

# 定义连接数据库的方法
def connect_to_postgresql(table_name):
    # 配置JDBC连接器所需的参数
    url = "jdbc:postgresql://your_postgresql_server:5432/your_database_name"
    properties = {
        "user": "your_username",
        "password": "your_password",
        "driver": "org.postgresql.Driver"
    }
    
    # 连接到数据库
    try:
      	sc = SparkContext()
        glue_Context = GlueContext(sc)
        spark = glue_Context.spark_session.builder.appName("PostgreSQLConnection").getOrCreate()
        df = spark.read.jdbc(url=url, table=table_name, properties=properties)
    except Exception as e:
        print("Error connecting to PostgreSQL database:", str(e))
        # 处理连接异常，例如记录日志或发出警报
        return None, None
    
    # 返回SparkSession对象和DataFrame对象
    return spark, df


# 定义关闭数据库连接的方法
def disconnect_from_postgresql(spark):
    # 关闭数据库连接
    try:
        spark.stop()
    except Exception as e:
        print("Error closing PostgreSQL database connection:", str(e))
        # 处理关闭连接异常，例如记录日志或发出警报
```



### 上传psycopg2包到aws上读写数据库，使用python3.10版本

- 通常，你可以下载以下文件来安装最新版本的psycopg2：

  psycopg2-3.0.3.tar.gz

  可以使用以下命令从文件安装psycopg2：

  ```python
  pip install psycopg2-3.0.3.tar.gz
  ```

- 如果没办法直接用pip安装，需要将psycopg2安装包下载到Windows电脑上，并将其上传到AWS S3存储桶以在AWS Glue作业中使用：

  对于Python 3.10和Windows操作系统，你可以下载以下文件：

  ```
  psycopg2-3.0.3-cp310-cp310-win_amd64.whl
  ```

  此外，如果你需要使用其他Python版本或操作系统，请在psycopg2的官方下载页面（https://pypi.org/project/psycopg2/#files）上查找适当的版本。

  请注意，你需要在AWS Glue环境中安装psycopg2的依赖项，例如libpq和其他必要的库。你可以在AWS Glue官方文档中找到有关如何在AWS Glue环境中安装和配置这些依赖项的更多信息。

