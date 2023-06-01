---
title: "常用的AWS CLI命令"
date: 2023-04-18T20:18:12+09:00
draft: false
categories:
- IT
tags:
- AWS
---



AWS Command Line Interface (CLI)是一个开源工具，它使你能够使用命令行接口（CLI）方式管理你的AWS服务。使用AWS CLI，你可以轻松地创建和管理AWS资源，例如EC2实例、S3存储桶、Lambda函数等。AWS CLI也提供了大量的选项和参数，可用于更精细的控制AWS服务。

AWS CLI基于Python开发，并且可以在Windows、Linux和MacOS等多种操作系统上运行。你可以使用pip等Python包管理器来安装AWS CLI，并且AWS CLI也内置在AWS提供的CloudShell中。

AWS CLI支持所有AWS服务，并且包含了所有AWS服务的API操作。使用AWS CLI可以自动化AWS资源的创建和配置，也可以通过Shell脚本或其他编程语言将其集成到自己的应用程序中。



---

#### 用AWS CLI命令启动EC2实例：

```shell
aws ec2 start-instances --instance-ids <instance-id>
```

将 `<instance-id>` 替换为你要启动的EC2实例的实例ID。

请注意，启动实例需要一些时间来完成，你可以使用以下命令来等待实例完全启动：

```
aws ec2 wait instance-running --instance-ids <instance-id>
```

这将一直等待，直到实例完全启动。



---

#### 用AWS CLI命令关闭EC2实例：

```
aws ec2 stop-instances --instance-ids <instance-id>
```

将 `<instance-id>` 替换为你要关闭的EC2实例的实例ID。

请注意，停止实例会将实例立即关闭，这可能会影响你的应用程序和用户。如果你希望避免意外关闭实例，请在执行此命令之前进行确认和备份。



---

#### 用AWS CLI命令启动AWS RDS数据库实例：

```
aws rds start-db-instance --db-instance-identifier <db-instance-id>
```

将 `<db-instance-id>` 替换为你要启动的RDS实例的实例ID。

请注意，启动数据库实例需要一些时间来完成，你可以使用以下命令来等待实例完全启动：

```
aws rds wait db-instance-available --db-instance-identifier <db-instance-id>
```

这将一直等待，直到数据库实例完全启动。



---

#### 用AWS CLI命令关闭AWS RDS数据库实例：

```
aws rds stop-db-instance --db-instance-identifier <db-instance-id>
```

将 `<db-instance-id>` 替换为你要关闭的RDS实例的实例ID。

请注意，停止数据库实例会将实例立即关闭，这可能会影响你的应用程序和用户。如果你希望避免意外关闭实例，请在执行此命令之前进行确认和备份。



---

#### 列出S3目录下的文件：

```
aws s3 ls s3://<bucket-name>/<path>
```

将 `<bucket-name>` 替换为你要列出文件的S3存储桶的名称，将 `<path>` 替换为你要列出文件的S3目录路径。如果要列出存储桶根目录下的所有文件，请省略 `<path>` 参数。



---

#### 将S3的文件复制到当前目录下：

```
aws s3 cp s3://<bucket-name>/<path>/<file> .
```

将 `<bucket-name>` 替换为存储桶的名称，将 `<path>` 替换为文件所在的S3目录路径，将 `<file>` 替换为要复制的文件的名称。注意，最后的 `.` 表示将文件复制到当前目录下。



---

#### 将S3的文件移动到另一个路径下：

```
aws s3 mv s3://<bucket-name>/<path>/<file> s3://<bucket-name>/<new-path>/<new-file>
```

将 `<bucket-name>` 替换为存储桶的名称，将 `<path>` 替换为文件所在的S3目录路径，将 `<file>` 替换为要移动的文件的名称，将 `<new-path>` 替换为新的S3目录路径，将 `<new-file>` 替换为移动后文件的名称。注意，此命令将文件从原始路径移动到新路径，如果新路径下存在同名的文件，它将被覆盖。



---

#### 使用AWS CLI命令通过SSH远程连接到EC2实例:

- 在AWS EC2控制台上获取你要连接的EC2实例的公共IP地址或DNS名称。

- 打开终端（在Windows上为命令提示符或PowerShell）。

- 在终端中，使用以下命令将.pem格式的密钥文件设置为仅限拥有者读取：

  ```shell
  chmod 400 <path-to-key-file>.pem
  ```

- 使用以下命令连接到EC2实例：

  ```shell
  ssh -i <path-to-key-file>.pem ec2-user@<public-ip-address-or-dns-name>
  ```

  将 `<path-to-key-file>` 替换为你的.pem格式密钥文件的路径，将 `<public-ip-address-or-dns-name>` 替换为你要连接的EC2实例的公共IP地址或DNS名称。

- 如果你使用的是Windows操作系统，则需要安装并使用SSH客户端，例如PuTTY，来远程连接到EC2实例。在这种情况下，你需要将.ppk格式的密钥文件转换为PuTTY可识别的格式。

