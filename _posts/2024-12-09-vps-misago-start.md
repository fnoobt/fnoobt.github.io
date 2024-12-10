---
title: Misago 安装
author: fnoobt
date: 2024-12-09 20:17:00 +0800
categories: [VPS,服务搭建]
tags: [vps,misago]
---

## 1. 更新系统包
```bash
sudo apt update
sudo apt upgrade -y
```

## 2. 安装必备依赖
Misago 需要 Python 和 PostgreSQL 数据库等支持，运行以下命令安装：
```bash
sudo apt install -y python3 python3-pip python3-venv \
    postgresql postgresql-contrib \
    libpq-dev git
```


## 3. 设置 PostgreSQL 数据库
### 3.1 切换到 PostgreSQL 管理账户：
```bash
sudo -i -u postgres
```

### 3.2 进入 PostgreSQL 命令行
```bash
psql
```

### 3.3 创建数据库和用户
将 `your_secure_password` 替换为你的安全密码。
```bash
CREATE DATABASE misago_db;
CREATE USER misago_user WITH PASSWORD 'your_secure_password';
GRANT ALL PRIVILEGES ON DATABASE misago_db TO misago_user;
```

### 3.4 退出 PostgreSQL
```bash
\q
```

### 3.5 返回普通用户
退出 postgres 用户：
```bash
exit
```

### 3.6 检查是否正确创建
回到普通用户后，你可以再次使用以下命令验证数据库和用户是否已正确创建：
```bash
sudo -i -u postgres psql -c "\l"
```

## 4. 克隆 Misago 源代码
从官方 GitHub 仓库获取最新代码：
```bash
git clone https://github.com/rafalp/Misago.git
cd Misago
```

## 5. 创建和激活虚拟环境
```bash
python3 -m venv venv
source venv/bin/activate
```

## 6. 安装 Python 依赖
使用 pip 安装 Misago 所需的依赖项：
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

## 7. 配置 Misago
编辑 Misago 配置文件以连接到你的 PostgreSQL 数据库：
```bash
vim misago/settings.py
```

在末尾添加以下内容：
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'misago_db',
        'USER': 'misago_user',
        'PASSWORD': 'your_secure_password',
        'HOST': 'localhost',  # 如果数据库运行在本地
        'PORT': '5432',       # 默认 PostgreSQL 端口
    }
}
```

> 注意：将 `your_secure_password` 替换为你的安全密码

## 8. 运行数据库迁移
```bash
python manage.py migrate
```

## 9. 创建管理员账户
```bash
python manage.py createsuperuser
```

此处需要按提示输入用户名、电子邮件和密码。

## 10. 收集静态文件
```bash
python manage.py collectstatic
```

## 11. 启动开发服务器
运行以下命令启动 Misago 的开发服务器：
```bash
python manage.py runserver
```

默认情况下，服务器会运行在 [http://127.0.0.1:8000](http://127.0.0.1:8000)。你可以通过浏览器访问它。

## 12. 使用生产环境部署
如果你需要在生产环境运行 Misago，建议使用 WSGI 服务器（如 Gunicorn）和反向代理服务器（如 Nginx）。


