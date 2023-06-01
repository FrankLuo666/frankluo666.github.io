---
title: "如何使用AWS Glue的工作流来指定job的调用顺序"
date: 2023-03-08T23:38:17+09:00
draft: false
categories:
- IT
tags:
- AWS
- AWS Glue
thumbnailImage: "https://d1u9biwaxjngwg.cloudfront.net/japanese-test-post/peak-140.jpg"
thumbnailImagePosition: left
autoThumbnailImage: yes
metaAlignment: center
summary: "“把字刻在石头上。”    -- 最古老的方式却是保留信息最长久的方式。"
---

### 关于AWS Glue Workflow

AWS Glue Workflow 是一个用于在 AWS Glue 中运行多个作业（Job）和其它任务的服务。Workflow 提供了一种创建、运行和监控多个作业和任务的方法，这些作业和任务可以按照指定的顺序执行。Workflow 还可以让你在作业或任务之间传递数据，并且支持跨多个账户和区域运行作业。

在 AWS Glue Workflow 中，你可以使用工作流设计器创建工作流，其中包括多个作业或任务，并且你可以指定这些作业或任务的执行顺序。例如，你可以创建一个工作流，其中包括一个作业，用于提取数据，然后将其输出传递给下一个作业，用于转换数据，并将其输出传递给第三个作业，用于加载数据到目标存储。Workflow 还提供了许多功能，例如错误处理和超时设置，以及支持传递参数和状态信息等功能。

Workflow 可以通过 AWS Glue 控制台、AWS CLI、AWS SDK 和 AWS CloudFormation 等方式进行管理和操作。通过 Workflow，你可以更轻松地实现数据处理和 ETL 等复杂工作流程，提高工作效率并降低错误率。



### 如何使用Workflow调用多个job，将上一个作业的输出作为参数传递给下一个作业

1. **创建工作流**

在 AWS Glue 控制台中，选择“工作流”，然后点击“创建工作流”。输入工作流名称和 IAM 角色，选择要使用的 Glue 数据库和 S3 输出路径。

2. **添加任务**

在工作流设计器中，选择“添加任务”，选择要调用的 AWS Glue 作业，并指定作业参数。例如，可以将上一个作业的输出作为参数传递给下一个作业：

```json
{
    "--input_path": "s3://your-bucket/path-to-input-file",
    "--output_path": "s3://your-bucket/path-to-output-file"
}
```

3. **连接任务**

在工作流设计器中，使用箭头将任务连接起来。这样，上一个作业的输出将会作为下一个作业的输入。

4. **运行工作流**

在工作流设计器中，点击“运行工作流”按钮。AWS Glue 将按照指定的顺序运行作业，并将上一个作业的输出作为下一个作业的输入。

在每个 Python 脚本中，需要将输出结果打印到 stdout，以便 AWS Glue 收集并传递到下一个作业。可以使用 `print()` 函数打印输出结果，然后使用 `sys.stdout.flush()` 函数强制将结果输出到 stdout。在下一个作业中，可以通过解析输入参数来获取上一个作业的输出结果。
