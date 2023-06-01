---
title: "AWS Glue概述及使用流程"
date: 2023-03-02T23:13:04+09:00
draft: false
categories:
- IT
tags:
- AWS
- AWS Glue
thumbnailImage: "https://d1u9biwaxjngwg.cloudfront.net/video-integration-showcase/peak-140.jpg"
thumbnailImagePosition: left
autoThumbnailImage: yes
metaAlignment: center
summary: "在中国，任何超脱飞扬的思想都会砰然坠地，因为现实的引力太沉重了。"
---



·

## 什么是AWS Glue ？

AWS Glue是一种完全托管的ETL（Extract, Transform, Load）服务，可用于处理和转换在AWS上存储的数据。AWS Glue自动扩展以适应大型数据集，同时为数据转换提供高效的方式。它可以与多个AWS数据存储服务集成，例如Amazon S3，Amazon RDS和Amazon Redshift等。使用AWS Glue，用户可以在不编写任何代码的情况下定义和运行ETL作业。

以下是使用AWS Glue的一些步骤：

1. 在AWS管理控制台中创建一个Glue数据目录。此操作可让用户访问其AWS Glue资源和作业。
2. 在Glue数据目录中创建一个数据库，以存储源数据。
3. 创建一个Glue爬行器（Crawler），以定义数据源的架构。AWS Glue会使用爬行器自动推断数据源的结构，并为数据生成一个表格定义。
4. 创建一个Glue作业（Job），以定义数据转换和目标存储。用户可以使用Glue作业将数据转换为目标格式，并将其加载到另一个数据存储中。
5. 调度Glue作业。用户可以根据需要调度Glue作业，以便自动运行转换作业。

总的来说，AWS Glue是一个强大的数据集成工具，可以帮助用户简化ETL工作流程，提高数据处理效率。



## 为什么要使用AWS Glue

AWS Glue是一种ETL（Extract-Transform-Load）服务，它可帮助您在不编写大量代码的情况下，轻松地准备和加载数据。以下是一些使用AWS Glue的好处：

1. 自动化：AWS Glue会自动扫描和抽取数据，以减少手动操作。
2. 降低成本：AWS Glue提供完全托管的服务，因此您无需担心维护基础架构，而只需支付实际使用的服务费用。
3. 弹性伸缩：AWS Glue能够自动扩展以处理大量数据，因此您无需担心集群规模的问题。
4. 灵活性：AWS Glue支持多种不同的数据源和数据格式，使您能够从多个来源提取和转换数据。
5. 集成性：AWS Glue与AWS的其他服务集成紧密，如Amazon S3、Amazon RDS和Amazon Redshift等，使数据处理更加流畅和高效。

总之，AWS Glue提供了一种简单、高效和弹性的方式来准备和加载数据，使您能够更快地分析和利用数据，从而实现更好的业务决策和增长。



## 如何使用AWS Glue ？

​    假设我需要用python从AWS s3 bucket的某个文件夹中读取其中的文件。文件中记载的是学生信息，有ID、姓名、年龄、住址、电话、年级等等信息。我需要用AWS Glue对其中的姓名，电话和住址进行一系列的统计分析，之后需要将结果储存到PostgreSQL数据库中。我该如何做呢？



### 1. 连接到S3存储桶并读取文件

​	首先，需要在AWS控制台中创建一个S3存储桶，并将包含学生信息的文件上传到其中。接下来，可以使用boto3库连接AWS S3并读取文件。以下是一个示例代码，假设S3存储桶名为"my-bucket"，文件夹名为"student-info"，文件名为"student.csv"：

```python
import boto3
import csv

# 创建S3资源对象
s3 = boto3.resource('s3')

# 定义S3存储桶的名称、文件夹名称和文件名称
bucket_name = 'my-bucket'
folder_name = 'student-info'
file_name = 'student.csv'

# 读取S3对象的内容
obj = s3.Object(bucket_name, f'{folder_name}/{file_name}')
body = obj.get()['Body'].read().decode('utf-8')

# 使用csv模块将文件内容分割为行
rows = csv.reader(body.split('\n'))

# 对于每一行，提取姓名、电话和住址信息，并执行统计分析
for row in rows:
    # 假设每行的信息用逗号分隔，第2列为姓名，第5列为电话，第4列为住址
    name = row[1]
    phone = row[4]
    address = row[3]

    # 在这里执行你的统计分析操作

# 保存结果到PostgreSQL数据库中
# TODO: 连接并写入数据到PostgreSQL数据库

```



### 2. 创建AWS Glue作业

```python
import boto3

# 创建AWS Glue客户端对象
glue = boto3.client('glue')

# 定义作业名称和作业脚本的S3位置
job_name = 'student_analysis'
script_location = 's3://my-bucket/glue_scripts/student_analysis.py'

# 创建AWS Glue作业
response = glue.create_job(
    Name=job_name,
    Role='AWSGlueServiceRoleDefault',
    Command={
        'Name': 'glueetl',
        'ScriptLocation': script_location
    },
    DefaultArguments={
        '--s3_source_bucket': bucket_name,
        '--s3_source_key': f'{folder_name}/{file_name}',
        '--database_name': 'student_db',
        '--table_name': 'student_analysis'
    },
    AllocatedCapacity=5,
    WorkerType='Standard'
)

job_id = response['JobName']
print(f'Job {job_id} created successfully.')

```



### 3. 将结果写入PostgreSQL数据库

```python
import psycopg2

# 连接到PostgreSQL数据库
conn = psycopg2.connect(
    host='my-database.host',
    port=5432,
    user='my-username',
    password='my-password',
    dbname='student_db'
)

# 创建一个新的游标对象
cur = conn.cursor()

# 插入数据到表中
cur.execute("INSERT INTO student_analysis (name, phone, address, analysis_result) VALUES (%s, %s, %s, %s)",
            (name, phone, address, analysis_result))

# 提交事务并关闭连接
conn.commit()
cur.close()
conn.close()

```

​	好了，今天就记到这里。
