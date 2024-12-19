---
title:  Misago 源码安装（Ubuntu 22.04）
author: fnoobt
date: 2024-12-09 20:17:00 +0800
categories: [VPS,服务搭建]
tags: [vps,linux,ubuntu,misago]
math: true
---

Misago 是一个用 Python 和 Django 开发的现代开源论坛（BBS）软件。它旨在提供强大、灵活、且用户友好的社区解决方案。

$$E = mc^2$$

\( x^2 + y^2 = z^2 \)

x^2

x^{10}

x_i^2

x^{\text{max}}

## 介绍
### Misago 的主要特性
1. 现代化设计：
 - 支持响应式设计，适配桌面和移动设备。
 - 界面简洁美观，符合当代设计风格。

2. 基于 Django 框架：
 - 使用 Django 开发，方便二次开发和与其他 Django 项目的集成。
 - 数据库操作和后台逻辑使用 Django 的 ORM 和框架机制。

3. 权限和角色管理：
 - 细粒度的权限系统，允许管理员设置不同用户组的访问权限。
 - 可为特定用户组或用户自定义权限。

4. 实时通知：
 - 提供类似社交平台的实时通知功能，让用户快速获知论坛动态。

5. 可扩展性：
 - 通过插件和自定义功能扩展论坛的功能。
 - 开发者可以使用 Django 的生态系统快速添加新功能。

6. 安全性：
 - 采用现代的安全技术，如 CSRF 防护、用户会话管理等。
 - 支持用户的密码加密和身份验证。

7. 国际化支持：
 - 内置多语言支持，可以方便地添加或切换语言。

### 适合的使用场景
- 社区论坛：搭建任何形式的在线社区，比如技术支持论坛、兴趣小组等。
- 内部讨论平台：企业内部团队的讨论工具。
- 替代传统论坛：需要迁移或升级到现代论坛平台的用户。

## 安装
### 1. 更新系统包
```bash
sudo apt update
sudo apt upgrade -y
```

### 2. 安装必备依赖
Misago 需要 Python 和 PostgreSQL 数据库等支持，运行以下命令安装：
```bash
sudo apt install -y python3 python3-pip python3-dev python3-venv \
    postgresql postgresql-contrib \
    build-essential libpq-dev libssl-dev libffi-dev git
```

> Misago 0.39.1 需要 python 3.11 或更高的版本
{: .prompt-warning }

```bash
# 查看 Python 版本时
python3 --version

# 安装 Python 3.11
sudo apt install python3.11 python3.11-venv python3.11-dev
```

### 3. 创建 Misago 用户
```bash
# 创建 misago 用户
sudo adduser misago_user

# 切换到 misago 用户
su - misago_user
```

> 将 `misago_user` 替换为你想创建的 Misago 用户名
{: .prompt-info }

之所以需要创建 Misago 用户，是因为 Misago 在配置文件中默认获取环境变量值，这些参数也可以手动修改。
```python
DATABASES = {
    "default": {
        # Misago requires PostgreSQL to run
        "ENGINE": "django.db.backends.postgresql",
        "NAME": os.environ.get("POSTGRES_DB"),
        "USER": os.environ.get("POSTGRES_USER"),
        "PASSWORD": os.environ.get("POSTGRES_PASSWORD"),
        "HOST": os.environ.get("POSTGRES_HOST"),
        "PORT": 5432,
    }
}
```
{: file='devproject/settings.py'}

### 4. 设置 PostgreSQL 数据库
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

> 将 `misago_user` 替换为你的系统用户名, 将 `your_secure_password` 替换为你的密码，两者需要与`devproject/settings.py`{: .filepath}中一致。
{: .prompt-info }

回到普通用户后，你可以使用以下命令验证数据库和用户是否已正确创建：
```bash
sudo -i -u postgres psql -c "\l"
```

### 5. 克隆 Misago 源代码
从官方 GitHub 仓库获取最新代码：
```bash
#下载整个项目
git clone https://github.com/rafalp/Misago.git
# 进入Misago目录
cd Misago
```

默认是开发版源码，可以选择切换到稳定的版本，比如0.39.1
```bash
git checkout 0.39.x
```

### 6. 创建和激活虚拟环境
在Misago目录下，为Misago创建一个虚拟环境，使Misago安装与系统上的其他Python包隔离。
```bash
# 如果默认是3.11及以上
python3 -m venv misagoenv
# 如果默认不是3.11及以上，强制指定
python3.11 -m venv misagoenv

# 激活虚拟环境
source misagoenv/bin/activate
```

### 7. 安装 Python 依赖
使用 pip 安装 Misago 所需的依赖项：
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### 8. 运行数据库迁移
```bash
python manage.py migrate
```

### 9. 创建管理员账户
```bash
python manage.py createsuperuser
```

此处需要按提示输入用户名、电子邮件和密码。

### 10. 更新静态文件
```bash
python manage.py collectstatic
```

### 11. 启动服务
运行以下命令启动 Misago 的服务：
```bash
python manage.py runserver 0.0.0.0:8000
```

> 添加`0.0.0.0:8000`作为参数，才可以实现局域网设备的访问
{: .prompt-warning }

默认情况下，Misago会运行在 `http://localhost:8000`。
管理员界面`http://localhost:8000/admincp/`。

### 12. 问题解决
#### 12.1 数据库迁移时提示`permission denied to create extension "hstore"`
解决方法：启用 `hstore` 扩展
```bash
# 切换到 PostgreSQL 管理账户, 并进入 PostgreSQL 命令行
sudo -u postgres psql

# 启用 hstore 扩展
CREATE EXTENSION IF NOT EXISTS hstore;
```

#### 12.2 数据库迁移时提示`permission denied to create extension "btree_gin"`
解决方法：启用 `btree_gin` 扩展
```bash
# 切换到 PostgreSQL 管理账户, 并进入 PostgreSQL 命令行
sudo -u postgres psql

# 启用 btree_gin 扩展
CREATE EXTENSION IF NOT EXISTS btree_gin;
```

#### 12.3 中文支持
Misago 从 0.39.1 开始原生支持简体中文，在`misago/locale/zh_Hans/LC_MESSAGES`{: .filepath}目录下存放`django.po`文件和编译生成的`djangojs.mo`文件。

Misago 默认通过获取环境变量选择系统使用的语言：
```python
LANGUAGE_CODE = os.environ.get("LANGUAGE_CODE", "") or "en-us"
```
{: file='devproject/settings.py'}

如果需要强制使用中文，需要修改为：
```python
LANGUAGE_CODE = 'zh-hans'
```
{: file='devproject/settings.py'}

此外设置时区等操作都在`devproject/settings.py`{: .filepath}目录下。

****

本文参考

> 1. [Misago 开源项目文件结构](https://blog.csdn.net/gitblog_01142/article/details/141076364)
> 2. [官方文档](https://misago.gitbook.io/docs/setup/misago)
> 3. [使用语言不是英文的问题](https://github.com/rafalp/Misago/issues/1426)