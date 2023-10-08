---
title: Debian10使用Rclone挂载Google云盘
author: fnoobt
date: 2021-11-09 20:21:00 +0800
categories: [VPS,服务搭建]
tags: [vps,rclone,google drive,linux]
---

通过Rclone挂载Google Drive无限容量的团队盘，也可以方便地将独服里下载的影片，传输到GD盘，此处简单记录下Rclone的设置挂载过程

## 安装Rclone

一条命令解决：

```bash
apt-get install wget -y && wget https://www.uud.me/usr/shell/rclone_debian.sh && bash rclone_debian.sh
```

## 配置克隆

安装完成后，配置rclone：

```bash
rclone config
```

```bash
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n #输入n
name> gdrive  #名称随便填
Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / 1Fichier
   \ "fichier"
 2 / Alias for an existing remote
   \ "alias"
 3 / Amazon Drive
   \ "amazon cloud drive"
 4 / Amazon S3 Compliant Storage Provider (AWS, Alibaba, Ceph, Digital Ocean, Dreamhost, IBM COS, Minio, etc)
   \ "s3"
 5 / Backblaze B2
   \ "b2"
 6 / Box
   \ "box"
 7 / Cache a remote
   \ "cache"
 8 / Citrix Sharefile
   \ "sharefile"
 9 / Dropbox
   \ "dropbox"
10 / Encrypt/Decrypt a remote
   \ "crypt"
11 / FTP Connection
   \ "ftp"
12 / Google Cloud Storage (this is not Google Drive)
   \ "google cloud storage"
13 / Google Drive
   \ "drive"
14 / Google Photos
   \ "google photos"
15 / Hubic
   \ "hubic"
16 / JottaCloud
   \ "jottacloud"
17 / Koofr
   \ "koofr"
18 / Local Disk
   \ "local"
19 / Mail.ru Cloud
   \ "mailru"
20 / Mega
   \ "mega"
21 / Microsoft Azure Blob Storage
   \ "azureblob"
22 / Microsoft OneDrive
   \ "onedrive"
23 / OpenDrive
   \ "opendrive"
24 / Openstack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
25 / Pcloud
   \ "pcloud"
26 / Put.io
   \ "putio"
27 / QingCloud Object Storage
   \ "qingstor"
28 / SSH/SFTP Connection
   \ "sftp"
29 / Transparently chunk/split large files
   \ "chunker"
30 / Union merges the contents of several remotes
   \ "union"
31 / Webdav
   \ "webdav"
32 / Yandex Disk
   \ "yandex"
33 / http Connection
   \ "http"
34 / premiumize.me
   \ "premiumizeme"
Storage> 13 #选择13，google drive
** See help for drive backend at: https://rclone.org/drive/ **

Google Application Client Id
Setting your own is recommended.
See https://rclone.org/drive/#making-your-own-client-id for how to create your own.
If you leave this blank, it will use an internal key which is low performance.
Enter a string value. Press Enter for the default ("").
client_id>  #留空
Google Application Client Secret
Setting your own is recommended.
Enter a string value. Press Enter for the default ("").
client_secret>   #留空
Scope that rclone should use when requesting access from drive.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / Full access all files, excluding Application Data Folder.
   \ "drive"
 2 / Read-only access to file metadata and file contents.
   \ "drive.readonly"
   / Access to files created by rclone only.
 3 | These are visible in the drive website.
   | File authorization is revoked when the user deauthorizes the app.
   \ "drive.file"
   / Allows read and write access to the Application Data folder.
 4 | This is not visible in the drive website.
   \ "drive.appfolder"
   / Allows read-only access to file metadata but
 5 | does not allow any access to read or download file content.
   \ "drive.metadata.readonly"
scope> 1 #输入1
ID of the root folder
Leave blank normally.

Fill in to access "Computers" folders (see docs), or for rclone to use
a non root folder as its starting point.

Note that if this is blank, the first time rclone runs it will fill it
in with the ID of the root folder.

Enter a string value. Press Enter for the default ("").
root_folder_id> #留空
Service Account Credentials JSON file path 
Leave blank normally.
Needed only if you want use SA instead of interactive login.
Enter a string value. Press Enter for the default ("").
service_account_file> 
Edit advanced config? (y/n)
y) Yes
n) No
y/n> n #输入n
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine
y) Yes
n) No
y/n> n  #输入n
If your browser doesn't open automatically go to the following link: https://accounts.google.com/o/oauth2/auth?access_type=offline&client_id=202264815644.apps.googleusercontent.com&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&response_type=code&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fdrive&state=tZzSXMvZc8iu0lYFB4ANBw
Log in and authorize rclone for access
Enter verification code>  #打开以上链接，登录要绑定的Google,此处输入活动的Code
Configure this as a team drive?
y) Yes
n) No
y/n> y   #如果你的是团队盘，则输入y，否则n
Fetching team drive list...
Choose a number from below, or type in your own value
 1 / 幽游地资源收集
   \ "0AMp-QeDCIy_mUk9PVA"
Enter a Team Drive ID> 1 #如果是团队盘，输入序号
--------------------
[gdrive]
type = drive
scope = drive
token = {"access_token":"ya29.Il-2B5z7OPWyx59ZHw7__IemaHsR8VT7P__jUN27hnNXZtaj0Rk1HcWPGGt2xqjkJH3e2KaVWuwz1nvW20MT0rfmEd5XAMY-je7wzQWgdjuGaBn9-txOUGh2jkk_CYio2w","token_type":"Bearer","refresh_token":"1//03yKvDZ7SOc80CgYIARAAGAMSNwF-L9Irrptm9elmrbRA1jxPeBsodd2FZw0TG5Utj5dUXeEDxKgZcR2EkYy1D0_wpsWsmgCWDzg","expiry":"2019-12-24T03:17:54.135381482-05:00"}
team_drive = 0AMp-QeDCIy_mUk9PVA
--------------------
y) Yes this is OK
e) Edit this remote
d) Delete this remote
y/e/d> y #输入y
Current remotes:

Name                 Type
====                 ====
gdrive               drive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q #绑定完成后输入q退出，如果要继续绑定，输入y重复以上步骤
```

## 挂载为磁盘

```bash
mkdir /root/GoogleDrive #创建用于挂载的路径 即下面的 LocalFolder
rclone mount DriveName:Folder LocalFolder --copy-links --no-gzip-encoding --no-check-certificate --allow-other --allow-non-empty --umask 000 #进行挂载，参数解释看下方
```

`DriveName`为初始化配置时随便填写的名称

`文件夹`为谷歌硬盘里的文件夹路径

`LocalFolder`为刚刚创建的目录

示例：

```bash
rclone mount gdrive: /root/GoogleDrive --copy-links --no-gzip-encoding --no-check-certificate --allow-other --allow-non-empty --umask 000
```

以上这个示例中，文件夹为空，表示挂载**Google云端硬盘**所有目录文件

输入`df -h`查看挂载的硬盘：

## 开机自启

新建一个rclone.service文件：

```bash
vi /usr/lib/systemd/system/rclone.service
```

撰写：

```markdown
[Unit]
Description = rclone

[Service]
User = root
ExecStart = /usr/bin/rclone mount gdrive: /root/GoogleDrive --copy-links --no-gzip-encoding --no-check-certificate --allow-other --allow-non-empty --umask 000
Restart = on-abort

[Install]
WantedBy = multi-user.target
```

重载daemon，让新的服务文件体现：

```bash
systemctl daemon-reload
```

设置开机启动：

```bash
systemctl enable rclone
```

停止，查看状态可以用：

```bash
systemctl stop rclone 
systemctl status rclone
```

****

> [原文出处](https://www.nbmao.com/archives/3765)
