---
title: "在linux中用curl命令下载音视频文件"
date: 2023-05-15T18:56:54+09:00
draft: false
categories:
- IT
tags:
- Linux
- 工具
---



#### curl命令简介

`curl` 是一个功能强大的命令行工具，用于与 URL 相关的各种操作，例如下载文件、发送请求、获取网页内容等。下面是一些 `curl` 命令的常用选项和用法：

基本用法：

```shell
curl <URL>
```

这将以 GET 请求方式获取指定 URL 的内容并将其输出到终端。

常用选项：

- `-o <文件名>`：将下载的内容保存到指定的文件中。
- `-O`：将下载的内容保存到与远程文件名相同的文件中。
- `-J`：使用远程服务器上的文件名保存下载的文件。
- `-L`：如果下载链接是重定向的，则跟随重定向。
- `-A <User-Agent>`：设置用户代理标头。
- `-H <Header>`：添加自定义标头。



#### 使用 `curl` 命令将音频/视频下载到指定目录并重命名

可以结合使用 `-L`  和 `-o`选项。

以下是将视频**下载到指定目录并重命名**的语法：

```shell
curl -L -o /path/to/newfilename.mp4 <下载链接>
```

其中：

- `-L`：跟随重定向。
- `-o /path/to/newfilename.mp4`：指定保存位置和重命名文件的路径。请将 `/path/to/newfilename.mp4` 替换为你希望视频保存的目录和新文件名。
- `<下载链接>`：你需要替换为实际的下载链接。

例如，用以下命令将音频下载到当前目录，并重命名为“夜曲.mp3”：

```shell
curl -L -o ./夜曲.mp3 https://s10.youtube4kdownloader.com/download7/mp3/99sbg8eloh/v0/a/c5/cdb1/cd6
```



#### 顺便提供一个可以解析youtube视频网址的网站：

https://youtube4kdownloader.com/





#### curl命令的其他用法：

发送 POST 请求：

```shell
curl -X POST -d "param1=value1&param2=value2" <URL>
```

发送带有 JSON 数据的 POST 请求：

```shell
curl -X POST -H "Content-Type: application/json" -d '{"key1":"value1", "key2":"value2"}' <URL>
```

上传文件：

```shell
curl -F "file=@/path/to/file" <URL>
```

使用代理服务器：

```shell
curl -x <proxy_address:port> <URL>
```

这只是 `curl` 命令的一些常见用法和选项示例。`curl` 还支持更多功能和选项，你可以通过查看 `curl` 的文档或运行 `man curl` 命令来获取更详细的信息。
