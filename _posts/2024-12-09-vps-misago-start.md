---
title:  Misago 源码安装（Ubuntu 22.04）
author: fnoobt
date: 2024-12-09 20:17:00 +0800
categories: [VPS,服务搭建]
tags: [vps,linux,ubuntu,misago]
---

## 1. 更新系统包
```bash
sudo apt update
sudo apt upgrade -y
```

## 2. 安装必备依赖
Misago 需要 Python 和 PostgreSQL 数据库等支持，运行以下命令安装：
```bash
sudo apt install -y python3 python3-pip python3-dev python3-venv \
    postgresql postgresql-contrib \
    build-essential libpq-dev libssl-dev libffi-dev git
```

>  注意：Misago 0.39.1 需要 python 3.11 或更高的版本

```bash
# 查看 Python 版本时
python3 --version

# 安装 Python 3.11
sudo apt install python3.11 python3.11-venv python3.11-dev
```

## 3. 设置 PostgreSQL 数据库
```bash
# 切换到 PostgreSQL 管理账户, 并进入 PostgreSQL 命令行，这两步可以分开
sudo -u postgres psql

# 创建数据库
CREATE DATABASE misago_db;

# 创建用户和密码
CREATE USER misago_user WITH PASSWORD 'your_secure_password';

# 授权用户管理数据库
GRANT ALL PRIVILEGES ON DATABASE misago_db TO misago_user;

# 退出 PostgreSQL
\q
```

> 注意：将 `misago_user` 替换为你的系统用户名, 将 `your_secure_password` 替换为你的密码。

回到普通用户后，你可以使用以下命令验证数据库和用户是否已正确创建：
```bash
sudo -i -u postgres psql -c "\l"
```

## 4. 克隆 Misago 源代码
从官方 GitHub 仓库获取最新代码：
```bash
#下载整个项目
git clone https://github.com/rafalp/Misago.git
# 进入Misago目录
cd Misago
```

## 5. 创建和激活虚拟环境
在Misago目录下，为Misago创建一个虚拟环境，使Misago安装与系统上的其他Python包隔离。
```bash
# 如果默认是3.11及以上
python3 -m venv misagoenv
# 如果默认不是3.11及以上，强制指定
python3.11 -m venv misagoenv

# 激活虚拟环境
source misagoenv/bin/activate
```

## 6. 安装 Python 依赖
使用 pip 安装 Misago 所需的依赖项：
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

## 7. 运行数据库迁移
```bash
python manage.py migrate
```

## 8. 创建管理员账户
```bash
python manage.py createsuperuser
```

此处需要按提示输入用户名、电子邮件和密码。

## 9. 收集静态文件
```bash
python manage.py collectstatic
```

## 10. 启动开发服务器
运行以下命令启动 Misago 的开发服务器：
```bash
# 指定0.0.0.0:8000，才可以实现其他设备的访问
python manage.py runserver 0.0.0.0:8000
```

默认情况下，Misago会运行在 [http://localhost:8000](http://localhost:8000)。
管理员界面[http://localhost:8000/admincp/](http://localhost:8000/admincp/)。


## 11. 问题解决
### 11.1 数据库迁移时提示`permission denied to create extension "hstore"`
解决方法：启用 hstore 扩展
```bash
# 切换到 PostgreSQL 管理账户, 并进入 PostgreSQL 命令行
sudo -u postgres psql

# 启用 hstore 扩展
CREATE EXTENSION IF NOT EXISTS hstore;
```

### 11.2 数据库迁移时提示`permission denied to create extension "btree_gin"`
解决方法：启用 hstore 扩展
```bash
# 切换到 PostgreSQL 管理账户, 并进入 PostgreSQL 命令行
sudo -u postgres psql

# 启用 btree_gin 扩展
CREATE EXTENSION IF NOT EXISTS btree_gin;
```