---
title: "如何使用AWS RDS创建Aurora PostgreSQL数据库、建表并进行增删改查等操作"
date: 2023-03-16T22:35:33+09:00
draft: false
categories:
- IT
tags:
- AWS
- Aurora
- postgreSQL
- sql
thumbnailImage: "https://d1u9biwaxjngwg.cloudfront.net/video-integration-showcase/peak-140.jpg"
thumbnailImagePosition: left
autoThumbnailImage: yes
metaAlignment: center
summary: "宇宙就是一座黑暗森林，每个文明都是带枪的猎人，像幽灵般潜行于林间，轻轻拨开挡路的树枝，竭力不让脚步发出一点儿声音，连呼吸都小心翼翼……他必须小心，因为林中到处都有与他一样潜行的猎人。如果他发现了别的生命，不管是不是猎人，不管是天使还是魔鬼，不管是娇嫩的婴儿还是步履蹒跚的老人，也不管是天仙般的少女还是天神般的男神，能做的只有一件事：开枪消灭之。在这片森林中，他人就是地狱，就是永恒的威胁，任何暴露自己存在的生命都将很快被消灭。这就是宇宙文明的图景，这就是对费米悖论的解释。"
---



### 什么是AWS Aurora

AWS Aurora是由亚马逊网络服务（AWS）提供的一种高性能关系型数据库服务，可以支持PostgreSQL和MySQL等开源数据库引擎，并提供与商业数据库引擎相当的性能和可靠性。

AWS Aurora是基于云原生架构设计的，它采用了分布式、高可用性和自我修复等技术，可以自动扩展以应对高负载和大规模数据存储需求。它还提供了多种数据备份和复制功能，可用于跨多个区域和可用区的数据冗余和恢复。

AWS Aurora的优势在于其高性能、可靠性和灵活性，可以在高并发、大数据和高可用性环境中实现快速和可靠的数据存储和访问。它还提供了与其他AWS服务和工具的紧密集成，可以轻松地集成到你的应用程序和工作流中。



### 如何在AWS RDS上创建Aurora PostgreSQL数据库

1. 登录AWS管理控制台并选择“RDS”服务。
2. 点击“创建数据库”按钮，然后选择“标准创建”选项。
3. 在“选择数据库引擎”页面上，选择“Aurora”作为数据库引擎，然后选择“Aurora PostgreSQL兼容性”作为数据库引擎版本。
4. 在“实例规格”页面上，选择你希望使用的实例规格，例如CPU、内存和存储容量等。
5. 在“设置”页面上，输入数据库实例的名称、用户名和密码等必要的信息，并设置你希望使用的其他选项，例如备份和监控。
6. 在“网络和安全性”页面上，选择你希望使用的VPC、子网和安全组等网络和安全设置。
7. 在“数据库选项”页面上，选择你希望使用的数据库选项，例如字符集和排序规则等。
8. 点击“创建数据库”按钮，等待数据库实例创建完成。

完成上述步骤后，你将能够使用AWS RDS创建Aurora PostgreSQL数据库。你可以使用AWS控制台、命令行界面或API来管理数据库实例并在其中创建数据库表、插入数据等操作。



### 如何设置安全组的入站规则，使其禁止外部IP访问

1. 在安全组页面上，找到你要修改的安全组并单击其ID。
2. 在“入站规则”标签下，单击“编辑入站规则”按钮。
3. 在“规则”下拉菜单中选择“自定义TCP规则”。
4. 在“协议”下拉菜单中选择“TCP”。
5. 在“端口范围”中输入你要允许的端口号，例如5432（对于PostgreSQL数据库）或3306（对于MySQL数据库）等。
6. 在“源”下拉菜单中选择“自定义”，然后输入EC2实例所在的安全组ID，例如sg-xxxxxxxxxxxx。
7. 单击“保存规则”按钮，保存更改。

这将允许来自EC2实例所在的安全组的任何IP地址的访问，但禁止所有其他IP地址的访问。请注意，如果你有多个EC2实例需要访问同一个数据库，你需要为每个EC2实例配置相同的安全组规则。



### 如何通过EC2访问PostgreSQL数据库

1.打开AWS管理控制台，并转到EC2服务页面。

2.选择你要访问数据库的EC2实例，并确保它已经与数据库实例在同一个VPC中。

3.安装你选择的PostgreSQL客户端，例如psql。

4.在EC2实例上打开命令行终端，并使用psql命令连接到PostgreSQL数据库。命令格式如下：

```shell
psql -h <database-endpoint> -p <port> -U <username> -d <database-name>
```

其中：

- `<database-endpoint>` 是你的数据库实例的终端节点地址。你可以在AWS控制台中找到它，格式为`<database-instance>.<random-string>.<region>.rds.amazonaws.com`。
- `<port>` 是数据库实例的端口号。默认情况下，PostgreSQL使用端口号5432。
- `<username>` 是你用于连接数据库的用户名。
- `<database-name>` 是你要连接的数据库名称。

例如，要连接到PostgreSQL数据库并使用用户名“myuser”连接到名为“mydb”的数据库，你可以运行以下命令：

```shell
psql -h mydb.c1234567890123456.us-east-1.rds.amazonaws.com -p 5432 -U myuser -d mydb
```

5.输入你的密码以登录到数据库。如果连接成功，你将看到一个提示符，如“mydb=>”。

现在你可以使用psql命令执行任何需要在PostgreSQL数据库上执行的操作，例如创建表、插入数据、查询数据等。



### 如何在PostgreSQL数据库中创建表，并进行增删改查等操作

假设我需要建一个user表，里面有id,name,gender,phoneNumber,address等字段，我该**如何建表：**

1.在EC2实例上，使用psql命令连接到PostgreSQL数据库。

2.输入以下命令来创建一个名为“user”的表：

```sql
CREATE TABLE user (
    id SERIAL PRIMARY KEY, #自增主键
    name VARCHAR(20),
    gender VARCHAR(4),
    phoneNumber VARCHAR(15),
    address VARCHAR(50)
);
```

3.输入以下命令来验证表是否创建成功：

```sql
SELECT * FROM user;
```

这将检索名为“user”的表中的所有行。由于表为空，此命令不会返回任何结果。

现在你已经成功创建了一个名为“user”的表，并定义了5个字段。你可以使用INSERT语句向表中插入数据，并使用SELECT语句从表中检索数据。



#### 如何向user表中插入数据：

1.在EC2实例上，使用psql命令连接到PostgreSQL数据库。

2.输入以下命令来向“user”表插入一条记录：

```sql
INSERT INTO user (name, gender, phoneNumber, address) VALUES ('John Doe', 'Male', '1234567890', '123 Main St.');
```

这将在“user”表中插入一行记录，包括名为“John Doe”的用户的姓名、性别、电话号码和地址。由于id列是自动增加的，你不需要指定它的值。

3.输入以下命令来验证数据是否已成功插入：

```sql
SELECT * FROM user;
```

这将检索名为“user”的表中的所有行。现在你应该可以看到一行记录，其中包含上一步插入的数据。

你可以重复这些步骤来插入更多的数据，或者使用批量插入操作来一次性插入多行数据。请注意，如果某个列是必填的（例如，id列是自动增加的主键），则必须为该列指定值或使用默认值。



#### 如何更改user表中的某一条数据：

1.在EC2实例上，使用psql命令连接到PostgreSQL数据库。

2.输入以下命令来更新名为“John Doe”的用户的电话号码：

```sql
UPDATE user SET phoneNumber = '9876543210' WHERE name = 'John Doe';
```

这将更新名为“John Doe”的用户的电话号码为“9876543210”。请注意，WHERE子句用于指定要更新的记录。如果省略WHERE子句，则将更新表中的所有记录。

3.输入以下命令来验证数据是否已成功更新：

```sql
SELECT * FROM user WHERE name = 'John Doe';
```

这将检索名为“John Doe”的用户的记录。现在，你应该可以看到该记录的phoneNumber列已更新为“9876543210”。

你可以重复这些步骤来更改其他记录。请注意，更新操作可能会影响多个记录，具体取决于WHERE子句。



#### 如何删除user表中的某一条数据：

1.在EC2实例上，使用psql命令连接到PostgreSQL数据库。



2.输入以下命令来删除名为“John Doe”的用户的记录：

```sql
DELETE FROM user WHERE name = 'John Doe';
```

这将从“user”表中删除名为“John Doe”的用户的记录。请注意，WHERE子句用于指定要删除的记录。如果省略WHERE子句，则将删除表中的所有记录。



3.输入以下命令来验证数据是否已成功删除：

  ```sql
  SELECT * FROM user;
  ```

  这将检索名为“user”的表中的所有行。现在，你应该不再看到名为“John Doe”的用户的记录。

  你可以重复这些步骤来删除其他记录。请注意，删除操作可能会影响多个记录，具体取决于WHERE子句。请确保在执行删除操作之前备份数据，并谨慎操作。
