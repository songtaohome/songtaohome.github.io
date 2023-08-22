---
layout: post
title: windows 下定时备份mysql
categories: Mysql
description: windows 下定时备份mysql
keywords: mysql, windows, 定时
---

在 Windows 服务器上定时备份 MySQL 数据库，只保存15天的备份，您可以使用 Windows 的任务计划程序来实现。以下是一步步的指南：

1. **编写备份脚本**：在您的 Windows 服务器上，创建一个 `.bat` 批处理文件，用于执行数据库备份操作。例如，创建一个名为 `backup_script.bat` 的文件，内容如下：

```bash
@echo off
setlocal

set BACKUP_DIR=C:\backup
for /f %%a in ('wmic os get LocalDateTime ^| find "."') do set datetime=%%a
set DATE=%datetime:~0,4%%datetime:~4,2%%datetime:~6,2%-%datetime:~8,2%%datetime:~10,2%%datetime:~12,2%
set MAX_BACKUP_AGE=15

"C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqldump" -u your_username -pYourPassword your_database_name > %BACKUP_DIR%\your_database_name_backup_%DATE%.sql

:: 删除超过 MAX_BACKUP_AGE 天的备份
forfiles /p %BACKUP_DIR% /m your_database_name_backup_* /d -%MAX_BACKUP_AGE% /c "cmd /c del @path"

endlocal

```



1. 将以下内容替换为实际信息：
   - `C:\backup`：您希望保存备份文件的目录路径。
   - `C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqldump`：`mysqldump` 工具的路径。这是 MySQL 数据库备份工具。
   - `your_username`：MySQL 数据库的用户名。
   - `YourPassword`：MySQL 数据库用户的密码。
   - `your_database_name`：要备份的数据库的名称。
2. **设置任务计划程序**：
   - 打开任务计划程序：按下 `Win + R` 键，输入 `taskschd.msc` 并按回车。
   - 在任务计划程序库中，右键点击 `任务计划程序库`，选择 `创建任务`。
   - 在 "常规" 标签下，输入任务名称，并勾选 "为不在用户登录时运行的任务配置此任务"。
   - 在 "触发器" 标签下，选择 "新建"，然后配置您希望执行备份的时间和频率。
   - 在 "操作" 标签下，选择 "新建"，在 "程序/脚本" 中输入 `C:\path\to\backup_script.bat`，并在 "起始于" 中输入您 `.bat` 脚本所在的目录路径。
   - 在 "条件" 标签下，您可以选择一些适当的条件，例如仅在计算机在电源插入时运行任务。
   - 确认并创建任务。
3. **测试任务**：您可以右键点击任务计划程序库中的任务，选择 "运行" 来测试任务是否正常运行。

现在，您已经设置了一个定时任务，每天将指定的 MySQL 数据库备份到指定的目录中。请确保您的备份目录具有足够的磁盘空间，并定期检查备份文件以确保备份正常运行。