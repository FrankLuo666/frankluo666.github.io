---
title: "在windows电脑中创建定时任务"
date: 2023-04-20T01:32:05+09:00
draft: false
categories:
- IT
tags:
- Windows
- bat脚本
---



#### .bat文件

`.bat` 是 Windows 操作系统中的批处理文件扩展名。批处理文件是一种包含一系列操作和命令的文本文件，可以由操作系统解释器（通常是命令提示符或 PowerShell）执行。

`.bat` 文件通常用于自动化重复性任务或简化复杂的操作序列，例如在系统启动时自动运行一组程序，或在特定时间执行特定任务。这些文件可以通过简单的文本编辑器创建，如 Windows 的记事本程序，你只需要在其中输入一系列命令并保存为 `.bat` 文件即可。

`.bat` 文件中可以包含很多不同的命令和操作，包括文件和目录管理、网络设置、系统配置等等。你还可以使用条件语句和循环来控制文件执行的流程，以及调用其他程序和脚本来扩展其功能。

总的来说，`.bat` 文件是一种强大的自动化工具，可以帮助简化日常任务和提高工作效率。



---

#### 创建bat文件，定时启动程序

打开记事本或任何文本编辑器，使用以下的命令来让它在系统时间的9:30和19:30启动程序：

```shell
@echo off

set "executed1=0"
set "executed2=0"

:loop
for /f "tokens=1-3 delims=: " %%a in ("%time%") do (
    set /a hours=%%a
    set /a minutes=%%b
    set /a seconds=%%c
)

if %executed1% equ 0 (
    if %hours% equ 9 if %minutes% geq 30 (
        start "" "<程序路径1>"
        set "executed1=1"
    )
)

if %executed2% equ 0 (
    if %hours% equ 19 if %minutes% geq 30 (
        start "" "<程序路径2>"
        set "executed2=1"
    )
)

if %executed1% equ 1 if %executed2% equ 1 (
    exit
)

timeout /t 30 /nobreak >nul
goto loop

```

上面的代码中，`set "executed1=1"` 和 `set "executed1=0"` 是设置环境变量 `executed1` 的值为 1 或 0。在代码中，我们使用这个变量来记录程序是否已经在当前时间段被执行过，以避免重复执行。具体来说，当程序1在 9:30 或 19:30 被执行时，`executed1` 被设置为 1，当它未被执行时，`executed1` 被设置为 0。同理，`executed2` 用于记录程序2是否被执行过。

使用`for`循环获取当前时间中的小时、分钟和秒数。然后检查小时和分钟。

如果小时等于 9 且分钟数大于或等于 30，则使用 `start` 命令启动程序 1，`executed1` 设置为 1。如果小时等于 19 且分钟数大于或等于 30，则使用 `start` 命令启动程序 2，`executed2` 设置为 1。

请将 `<程序路径1>` 和 `<程序路径2>` 分别替换为你要启动的程序 1 和程序 2 的路径。如果程序路径包含空格，请将其用引号括起来。

当`executed1` 和`executed2` 都被设置为 1时，表示两个程序都启动了，用exit退出程序。

将上述代码保存为 `.bat` 文件，例如 `start_programs.bat`。然后，你可以使用 Windows 任务计划程序来在每天 9:30 和 19:30 启动该 `.bat` 文件。



---

#### 每天定时执行.bat文件

要使 `.bat` 文件每天都运行，可以使用 Windows 任务计划程序来安排任务。这样，你可以指定每天在特定时间运行你的 `.bat` 文件。

下面是在 Windows 任务计划程序中安排任务的步骤：

1. 打开“任务计划程序”应用程序。
2. 在左侧窗格中，单击“创建任务”。
3. 在“常规”选项卡上，输入任务的名称和描述，然后选择“配置为”Windows 10”。
4. 在“触发器”选项卡上，单击“新建”。
5. 在“新建触发器”对话框中，选择“每天”，然后指定运行任务的时间。
6. 在“操作”选项卡上，单击“新建”。
7. 在“新建操作”对话框中，选择“启动程序”选项卡，然后指定要运行的 `.bat` 文件的路径和名称。
8. 单击“确定”以保存操作和任务。

现在，Windows 将在每天指定的时间运行你的 `.bat` 文件。
