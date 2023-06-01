---
title: "在linux中将文件压缩并加密"
date: 2023-05-22T22:50:20+09:00
draft: false
categories:
- IT
tags:
- linux
- shell脚本
---



#### 使用`zip`命令

`zip`是一个常用的命令行工具，用于创建和管理.zip压缩文件。它可以将文件和文件夹压缩为一个或多个.zip文件，以便于存储、传输和共享。

以下是一些常见的`zip`命令选项和用法：

1、创建压缩文件：

```shell
zip archive.zip file1.txt file2.txt folder/
```

这将创建一个名为`archive.zip`的压缩文件，并将`file1.txt`、`file2.txt`以及`folder`文件夹中的所有内容添加到压缩文件中。

2、 递归压缩文件夹：

```shell
zip -r archive.zip folder/
```

使用`-r`选项，可以递归地将文件夹`folder`及其所有内容添加到压缩文件中。

3、 添加新文件到现有压缩文件：

```shell
zip archive.zip newfile.txt
```

这将向现有的`、rchive.zip`压缩文件中添加`newfile.txt`。

4、解压缩压缩文件：

```shell
unzip archive.zip
```

这将解压缩名为`archive.zip`的压缩文件。

5、 显示压缩文件中的文件列表：

```shell
zipinfo archive.zip
```

使用`zipinfo`命令，可以查看压缩文件`archive.zip`中的文件列表和详细信息。

---



#### 使用`zip`命令设置密码并压缩文件夹

可以按照以下方式编写命令：

```shell
zip -er folder.zip folder/
```

在这个命令中，`-e`选项用于启用加密功能，`-r`选项用于递归地压缩文件夹，`folder.zip`是你要创建的压缩文件的名称，`folder/`是要压缩的文件夹的路径。

执行上述命令后，`zip`会提示你输入密码并确认密码。请确保选择一个强密码，并确保牢记它，因为以后解压缩文件时需要输入相同的密码。

---



#### 批量压缩文件夹下的mp3和mp4文件并设置密码

可以使用以下的shell脚本。在Linux中保存为.sh文件运行，在windows电脑中可以用git执行.sh文件，在MAC电脑中将后缀改为command就能直接运行。

```shell
#!/bin/sh

# 设置工作目录。将压缩此目录下的mp3和mp4文件。
folder="/Users/name/Documents/vedio/test"
cd "$folder"

# 保存IFS并设置为换行符
oldifs="$IFS"
IFS=$'\n'

# 读取密码
password_file="/Users/name/Documents/vedio/password.txt"
password=$(cat "$password_file")

for file in `find ./ -type f \( -iname \*.mp3 -o -iname \*.mp4 \)`
do
    if test -f "$file"
    then
	zipName=${file%%[*}
	zipName=${zipName%.mp3}
	zipName=${zipName%.mp4}
	echo 加密并压缩 $file
	zip -e -P "$password" $zipName.zip $file
	echo 删除 $file
	rm -f $file
	echo -----------------------------
    fi
done
IFS="$oldifs"
echo 加密并压缩完毕，请关闭此窗口。
```

请确保将"/Users/name/Documents/vedio/test"替换为实际需要进行压缩的文件所在的文件夹，将`/Users/name/Documents/vedio/password.txt`替换为实际的密码文件路径。

---



#### 批量解压文件夹下设有密码的压缩文件

对应上面的压缩文件的解压方法。

可以使用以下shell脚本。在Linux中是.sh文件，在windows电脑中可以用git执行.sh文件，在mac电脑中将后缀改为command就能直接运行。

```shell
#!/bin/sh

# 设置工作目录。将解压此目录中的zip压缩文件。
folder="/Users/name/Documents/vedio/test"
cd "$folder"

# 保存IFS并设置为换行符
oldifs="$IFS"
IFS=$'\n'

# 读取密码
password_file="/Users/name/Documents/vedio/password.txt"
password=$(cat "$password_file")
echo "$password"

for file in `find . -name "*.zip"`
do
    if test -f "$file"
    then
	echo 解压 $file
	unzip -o -P "$password" $file
	echo 删除 $file
	rm -f $file
	echo -----------------------------
    fi
done
echo 解压完毕，请关闭此窗口。
```

请确保将`/Users/name/Documents/vedio/test`替换为实际需要进行解压的文件所在的文件夹，将`/Users/name/Documents/vedio/password.txt`替换为实际的密码文件路径。

