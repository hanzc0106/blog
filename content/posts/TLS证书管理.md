+++
date = '2025-03-29T13:35:39+08:00'
draft = false
title = 'TLS证书管理'
tags = [网站开发, 网站部署, TLS, TLS证书]
categories = [网站开发, 网站部署, TLS, TLS证书]
+++

# TLS证书管理

使用`Let's Encrypt`申请和部署免费的`TLS`证书，并实现自动续期。

## 申请证书

### 1. 安装`acme.sh`

```bash
# 安装 acme.sh
wget -O -  https://get.acme.sh | sh
# 使 acme.sh 生效
. .bashrc
# 配置 acme.sh 自动升级
acme.sh --upgrade --auto-upgrade
```

### 2. 测试申请证书

```bash
acme.sh --issue --server letsencrypt --test -d 二级域名.你的域名.com -w /var/www/二级域名.你的域名.com --keylength ec-256
```

如果出错了，在最加上`--debug`参数，查看详细错误信息。

```bash
acme.sh --issue --server letsencrypt --test -d 二级域名.你的域名.com -w /var/www/二级域名.你的域名.com --keylength ec-256 --debug
```

### 3. 申请证书
```bash
# 配置默认 CA 为 letsencrypt
acme.sh --set-default-ca --server letsencrypt
# 申请证书，使用 --force 强制覆盖旧证书
acme.sh --issue -d 二级域名.你的域名.com -w /var/www/二级域名.你的域名.com --keylength ec-256 --force
```

### 4. 安装证书
```bash
acme.sh --installcert -d 二级域名.你的域名.com --cert-file /你要安装到的位置/cert.crt --key-file /你要安装到的位置/cert.key --fullchain-file /你要安装到的位置/fullchain.crt --ecc
```

### 5. 自动续期

#### 5.1 自动续期脚本
建立脚本文件`/home/你的用户名/cert/cert-renew.sh`，内容如下：
```bash
#!/bin/bash

/home/你的用户名/.acme.sh/acme.sh --install-cert -d 二级域名.一级域名.com --ecc --fullchain-file /home/你的用户名/cert/fullchain.crt --key-file /home/你的用户名/cert/cert.key
echo "Certificates Renewed"

chmod +r /home/你的用户名/cert/cert.key
echo "Read Permission Granted for Private Key"

sudo systemctl restart nginx
echo "nginx Restarted"
```

TLS证书Nginx使用场景较多，所以需要配置cert.key的权限为可读, 并重启nginx服务。

给这个脚本增加【可执行】权限：
```bash
chmod +x cert-renew.sh
```

#### 5.2 配置定时任务
```bash
# 编辑定时任务
crontab -e
```
在文件中添加以下内容：
```bash
# 1:00am, 1st day each month, run `cert-renew.sh`
0 1 1 * *   bash /home/你的用户名/cert/cert-renew.sh
```