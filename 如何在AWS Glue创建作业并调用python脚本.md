---
title: "如何在AWS Glue创建作业并调用python脚本"
date: 2023-03-08T23:14:51+09:00
draft: false
categories:
- IT
tags:
- AWS Glue
- python
thumbnailImage: "https://d1u9biwaxjngwg.cloudfront.net/chinese-test-post/vintage-140.jpg"
thumbnailImagePosition: left
autoThumbnailImage: yes
metaAlignment: center
summary: "给时光以生命，而不是给生命以时光。"
---



### 关于AWS Glue

AWS Glue是一个完全托管的ETL（Extract, Transform, Load）服务，可用于准备和加载数据以进行分析、机器学习和应用程序开发。它可以自动发现和分类数据，为你提供ETL作业的自动编排和扩展能力，并在AWS各种服务之间提供无缝连接，例如Amazon S3，Amazon RDS和Amazon Redshift。

AWS Glue提供了一种可扩展的服务来编写和执行ETL作业，可以使用Python或Scala编写ETL脚本，也可以使用可视化工具来创建和管理作业。AWS Glue还提供了一个数据目录，可用于发现、搜索和查询各种数据资源和元数据，包括数据库表、S3对象和Kafka主题等。

AWS Glue还提供了许多与安全性、可扩展性、可靠性和成本效益有关的特性，例如自动扩展、负载均衡、安全数据访问、数据加密、监控和日志记录等。

###     如何在aws glue中创建一个job，并在其中调用python脚本去读取S3里的csv文件？

1. 登录 AWS Glue 控制台：https://console.aws.amazon.com/glue/

2. 在导航菜单中，选择“作业”，然后选择“添加作业”。

3. 在“添加作业”页面，输入作业名称，并选择要使用的 IAM 角色。此角色应具有访问 S3 和 Glue 的权限。

4. 在“数据源”选项卡下，选择“S3”作为数据源类型，然后输入 S3 桶和前缀的路径。

5. 在“转换”选项卡下，选择“Python Shell”作为作业类型，并输入要执行的 Python 脚本的路径。例如：s3://your-bucket-name/path-to-your-script.py

6. 在“目标”选项卡下，选择“创建表”。

7. 点击“提交作业”按钮。

   

   AWS Glue 会自动创建一个 Spark 环境，并在其中执行你的 Python 脚本。你的脚本可以使用 AWS Glue 的 PySpark API 读取和处理数据。

   

### 在aws gule中的job中如何向被调用的python脚本中传入参数，又如何获取python脚本的返回值呢？

1. **向 Python 脚本传递参数**

在 AWS Glue 的作业中向 Python 脚本传递参数，可以使用脚本参数功能。具体步骤如下：

- 在作业编辑器中，选择“转换”选项卡，在“脚本参数”部分中添加参数，例如：`--input_path` 和 `--output_path`。

- 在 Python 脚本中，可以使用 argparse 模块解析这些参数。例如：

  ```python
  import argparse
  
  parser = argparse.ArgumentParser()
  parser.add_argument("--input_path", help="input S3 path")
  parser.add_argument("--output_path", help="output S3 path")
  args = parser.parse_args()
  
  ```

  

2. **创建工作流**

在 AWS Glue 中获取 Python 脚本的返回值，可以使用 `JobRun` 对象。具体步骤如下：

- 在 Python 脚本中，使用 `print()` 函数输出需要返回的结果。例如：

```python
print("result: " + str(result))
```

- 在作业编辑器中，选择“目标”选项卡，勾选“收集作业输出”，并设置 S3 输出路径。
- 在 Python 脚本执行完成后，可以使用以下代码获取输出结果：

```python
pythonCopy code
import sys
import json

for line in sys.stdin:
    result = json.loads(line)
    print("result: " + str(result))
```

其中，`sys.stdin` 表示从 S3 中读取输出结果。输出结果是一个 JSON 字符串，需要使用 `json.loads()` 函数解析成 Python 对象。





### AWS Glue的一个job中，具体如何调用多个python shell？

1. 在 AWS Glue 控制台上，创建一个新的 Job，或编辑一个已有的 Job。
2. 在 Job 参数中，指定要执行的 Python Shell Job 脚本。你可以在“脚本路径”参数中指定一个或多个 Python 脚本的路径，以逗号分隔，如下所示：

```vbscript
s3://your-bucket/path-to-script1.py, s3://your-bucket/path-to-script2.py
```

1. 你还可以指定其他 Job 参数，例如输入输出路径、连接、安全配置等。在 Python Shell Job 脚本中，可以使用这些参数来访问和处理数据。
2. 运行 Job。AWS Glue 将按顺序执行指定的 Python Shell Job 脚本，并将输出传递给下一个脚本。如果任何一个脚本失败，整个 Job 将被中止。

需要注意的是，AWS Glue Job 中每个 Python Shell Job 脚本都是独立运行的，它们之间不共享变量和状态。如果需要在不同的 Python 脚本之间传递数据和状态，你可以使用 S3、DynamoDB、SQS 等 AWS 服务来实现。

### 举例说明同一个job中如何调用多个python shell：

​    假设我的S3里有4个python shell，分别执行简单的加减乘除4种运算。我要在同一个job中调用这4个python shell，它们按顺序分别被调用，它们输出的结果要储存到S3中。从第2个shell开始，每个shell都会接受上一个shell的运算结果作为参数，去进行它的运算。如何创建job，编写及调用python shell呢？

#### 方法一：

1. 在 S3 中创建 4 个 Python 脚本，分别实现加、减、乘、除运算，并将它们上传到 S3 桶中。

2. 在 AWS Glue 控制台上创建一个新的 Job。在 Job 参数中指定以下参数：

   a. `--input_path`：输入路径，指向包含输入数据的 S3 文件路径。

   b. `--output_path`：输出路径，指向存储输出数据的 S3 文件路径。

   c. `--operator`：运算符，指定要执行的运算类型。在第一个 Python 脚本中，将该参数设置为默认值 `add`。

   d. `--result`：结果，指定上一个 Python 脚本的输出结果。在第一个 Python 脚本中，将该参数设置为空字符串。

3. 在 Job 定义中指定 4 个 Python 脚本的路径和名称，以按顺序执行它们。

4. 在每个 Python 脚本中，读取输入数据，并根据指定的运算符和上一个 Python 脚本的输出结果，计算出当前 Python 脚本的输出结果，并将其保存到 S3 中指定的输出路径中。

   

   **示例代码 ：**

   add.py ：

   ```python
   import sys
   import boto3
   import json
   
   input_path = sys.argv[1]
   output_path = sys.argv[2]
   operator = sys.argv[3]
   result = sys.argv[4]
   
   s3 = boto3.client('s3')
   
   if result:
       result = int(result)
   else:
       result = 0
   
   if operator == "add":
       data = s3.get_object(Bucket='my-bucket', Key=input_path)['Body'].read().decode('utf-8')
       data = json.loads(data)
       result += sum(data)
   
   s3.put_object(Bucket='my-bucket', Key=output_path, Body=str(result).encode('utf-8'))
   
   ```

   sub.py :

   ```python
   import sys
   import boto3
   import json
   
   input_path = sys.argv[1]
   output_path = sys.argv[2]
   operator = sys.argv[3]
   result = sys.argv[4]
   
   s3 = boto3.client('s3')
   
   if result:
       result = int(result)
   else:
       result = 0
   
   if operator == "sub":
       data = s3.get_object(Bucket='my-bucket', Key=input_path)['Body'].read().decode('utf-8')
       data = json.loads(data)
       result -= sum(data)
   
   s3.put_object(Bucket='my-bucket', Key=output_path, Body=str(result).encode('utf-8'))
   
   ```

   mul.py :

   ```python
   import sys
   import boto3
   import json
   
   input_path = sys.argv[1]
   output_path = sys.argv[2]
   operator = sys.argv[3]
   result = sys.argv[4]
   
   s3 = boto3.client('s3')
   
   if result:
       result = int(result)
   else:
       result = 1
   
   if operator == "mul":
       data = s3.get_object(Bucket='my-bucket', Key=input_path)['Body'].read().decode('utf-8')
       data = json.loads(data)
       for d in data:
           result *= d
   
   s3.put_object(Bucket='my-bucket', Key=output_path, Body=str(result).encode('utf-8'))
   
   ```

   del.py :

   ```python
   ### 根据以上类推
   ```

   在AWS Glue的job中，我们需要按照以下步骤调用这四个Python脚本：

   1. 在job中添加四个Python Shell作业节点。
   2. 将第一个Python脚本设置为作业节点的入口点，并将S3输入文件路径作为其输入参数。同时将S3输出文件路径传递给第二个Python脚本。
   3. 将第二个Python脚本设置为第一个Python脚本的后继节点，并将第一个Python脚本的输出文件路径作为其输入参数。同时将S3输出文件路径传递给第三个Python脚本。
   4. 将第三个Python脚本设置为第二个Python脚本的后继节点，并将第二个Python脚本的输出文件路径作为其输入参数。同时将S3输出文件路径传递给第四个Python脚本。
   5. 将第四个Python脚本设置为第三个Python脚本的后继节点，并将第三个Python脚本的输出文件路径作为其输入参数。

   以上步骤可以在AWS Glue Console中完成。具体而言，需要在job编辑器中按照上述步骤添加作业节点，设置输入输出路径和参数，以及将作业节点连接起来。最后保存并运行job即可。

#### 	方法二：

​	可以使用AWS Glue的Python Shell作业类型。下面是详细步骤：

1. 将4个python shell脚本上传到S3中，并记住它们的S3路径。

2. 创建一个Python Shell作业，选择要使用的Python版本，并将作业类型设置为“Spark”。

3. 在作业参数中添加一个名为“s3_output_path”的参数，用于指定输出结果要存储到的S3路径。

4. 在“脚本路径”下添加以下代码，以调用四个Python脚本：

   ```python
   import subprocess
   
   # 调用第一个python脚本，不传递参数
   subprocess.call(['python', '/path/to/first/script.py'])
   
   # 调用第二个python脚本，传递第一个脚本的输出结果作为参数
   output_of_script1 = # 从S3获取第一个脚本的输出结果
   subprocess.call(['python', '/path/to/second/script.py', str(output_of_script1)])
   
   # 调用第三个python脚本，传递第二个脚本的输出结果作为参数
   output_of_script2 = # 从S3获取第二个脚本的输出结果
   subprocess.call(['python', '/path/to/third/script.py', str(output_of_script2)])
   
   # 调用第四个python脚本，传递第三个脚本的输出结果作为参数
   output_of_script3 = # 从S3获取第三个脚本的输出结果
   subprocess.call(['python', '/path/to/fourth/script.py', str(output_of_script3)])
   ```

5. 在“Python依赖项”下添加任何需要的依赖项。
6. 单击“完成”以保存作业。
7. 运行作业并等待作业完成。在作业完成后，可以在S3中找到输出结果。

- 注意：在第2个Python脚本及其后续脚本中，需要编写代码来接收前一个脚本的输出结果，并在进行下一步运算时将其用作参数。



​	以上第4步中具体如何获第一个脚本的输出结果并将其存储到变量`output_of_script1`中，示例：

```python
import boto3
import os

# 创建S3客户端
s3 = boto3.client('s3')

# 指定存储第一个脚本输出结果的S3对象的路径
s3_output_path = 's3://my-bucket/path/to/output_of_script1'

# 在本地创建一个文件，用于存储从S3下载的输出结果
local_output_path = os.path.join(os.environ['GLUE_TMP_DIR'], 'output_of_script1')
s3.download_file(Bucket='my-bucket', Key=s3_output_path, Filename=local_output_path)

# 读取文件内容，并存储到变量output_of_script1中
with open(local_output_path, 'r') as f:
    output_of_script1 = f.read().strip()

```

​		在上面的代码中，假设输出文件已经存储在名为`my-bucket`的S3存储桶中，并在路径`/path/to/output_of_script1`下。`os`模块用于操作本地文件路径。`GLUE_TMP_DIR`是AWS Glue在执行过程中提供的环境变量，指向本地临时文件夹的路径。`strip()`函数用于删除字符串开头和结尾的空白字符。
