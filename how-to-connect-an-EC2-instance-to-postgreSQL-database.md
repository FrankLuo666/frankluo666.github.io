---
title: "如何使用EC2的终端连接aws RDS上的Aurora PostgreSQL数据库"
date: 2023-03-26T22:07:50+09:00
draft: false
categories:
- IT
tags:
- AWS
- EC2
- RDS
thumbnailImage: "https://d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg"
thumbnailImagePosition: left
autoThumbnailImage: yes
metaAlignment: center
summary: "给岁月以文明，而不是给文明以岁月。"
---



### 用EC2连接AWS RDS上的Aurora PostgreSQL数据库：

1. 确保你的EC2实例和RDS数据库在同一个VPC（Virtual Private Cloud）中。如果不在同一个VPC中，你需要通过VPC peering或公共Internet连接连接它们。

2. 确保你的EC2实例已经安装了适当的PostgreSQL客户端。你可以使用以下命令在EC2上安装：

   ```
   sudo apt-get update
   sudo apt-get install postgresql-client
   ```

3. 确保你的RDS数据库已经设置了公共可访问性。在RDS控制台中选择你的Aurora PostgreSQL数据库，然后在“Connectivity & security”选项卡下检查“Public accessibility”是否为“Yes”。

4. 获取你的RDS数据库终端节点的终端点地址。你可以在RDS控制台的“Connectivity & security”选项卡下找到它。

5. 在EC2上使用以下命令连接到你的RDS数据库：

   

   ```shell
   psql -h <RDS终端节点的终端点地址> -U <你的数据库用户名> -d <你的数据库名称>
   
   #　示例命令：
   psql -h mydatabase.123456789012.us-east-1.rds.amazonaws.com -U myuser -d mydatabase
   ＃　根据提示输入数据库密码即可
   ```



### 如何确保EC2实例和RDS数据库在同一个VPC

如果你还没有创建EC2实例，你需要确保在创建EC2实例时选择的VPC与你已经创建的RDS实例所在的VPC相同。如果你已经创建了EC2实例，则可以通过以下步骤确定它是否在相同的VPC中：

1. 在AWS管理控制台中，导航到“EC2”服务。
2. 选择你的EC2实例，然后查看它的“Details”选项卡。
3. 在“Details”选项卡中，找到“Network”属性。其中包括该实例所在的VPC ID。
4. 找到你的RDS实例，然后查看它的“Connectivity & security”选项卡下的“VPC ID”属性。
5. 如果两个ID相同，则表示你的EC2实例和RDS实例在同一个VPC中。

如果你的EC2实例和RDS实例不在同一个VPC中，则需要创建一个VPC peering连接或者使用公共Internet连接它们。注意，如果你选择使用公共Internet连接，请确保适当地保护你的数据库，并使用适当的安全组和网络访问控制列表来控制访问。



### 如何创建VPC peering连接来连接EC2实例和RDS实例

1. 在AWS管理控制台中，导航到“VPC”服务。
2. 在左侧菜单中，选择“Peering Connections”。
3. 单击“Create Peering Connection”按钮，然后输入以下信息：
   - 名称标记（Name Tag）：一个描述性的名称，以帮助你识别此VPC peering连接。
   - VPC请求者（VPC Requester）：选择包含你的EC2实例的VPC。
   - VPC接受者（VPC Acceptor）：选择包含你的RDS实例的VPC。
   - 自动接受（Auto Accept）：选择“不自动接受”，以便你可以检查并手动接受VPC peering连接请求。
4. 单击“Create Peering Connection”按钮，然后AWS将为你创建一个新的VPC peering连接。
5. 你需要在VPC接受者VPC的路由表中添加一条路由，以允许来自VPC请求者VPC的流量通过VPC peering连接访问RDS实例。为此，请按照以下步骤操作：
   - 在AWS管理控制台中，导航到“VPC”服务，然后选择VPC接受者VPC。
   - 在左侧菜单中，选择“路由表”。
   - 找到VPC接受者VPC的主路由表，并单击它。
   - 在下方的“路由表详细信息”窗格中，单击“编辑”按钮。
   - 在“目标”字段中，输入EC2实例所在的CIDR块，并在“目标”下拉列表中选择VPC peering连接的ID。
   - 单击“保存”按钮。
6. 你现在可以在你的EC2实例上使用相同的方式连接到你的Aurora PostgreSQL数据库，这将与它们在同一个VPC中时一样。

请注意，VPC peering连接可以跨越不同的AWS账户。如果你的EC2实例和RDS实例不在同一个AWS账户中，则需要在VPC peering连接请求中指定目标AWS账户ID，并请求对方AWS账户的确认。

