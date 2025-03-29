+++
date = '2025-03-29T14:06:49+08:00'
draft = false
title = 'Nginx配置'
tags = [网站开发, 网站部署, Nginx, Nginx配置]
categories = [网站开发, 网站部署, Nginx, Nginx配置]
+++

# Nginx配置

## 通过 IP 访问网站

通过IP访问网站时，只能通过http协议访问，不能通过https协议访问。

```
server {
	listen 8080;
	server_name xxx.xxx.xxx.xxx;	# 服务器IP

	# 根路径与静态文件
	root /var/www/hanzc.fun;
	index index.html index.htm;
	location / {
		try_files $uri $uri/ /index.html;
		autoindex off;
	}

	# API 代理
	location /api {
		proxy_pass http://xxx.xxx.xxx.xxx:yyyy; # 后端服务器IP和端口
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded-Proto $scheme; # 告知后端是 HTTPS
	}

	# 静态文件缓存
	location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
		try_files $uri =404;
		expires 1y;
		add_header Cache-Control "public, immutable";
	}

	# 日志
	access_log /var/log/nginx/hanzc.fun.access.log;
	error_log /var/log/nginx/hanzc.fun.error.log;
}
```

## 通过域名访问网站

通过域名访问网站时，一般只允许通过`https`协议访问。
将`http`协议的请求重定向到`https`协议的请求。

```
server {
	listen 80;
	server_name a-domain.com;
	return 301 https://$host$request_uri;
}

server {
	listen 443 ssl http2;
	server_name a-domain.com;

	# SSL 证书
	ssl_certificate /xxx-path-to-cert-xxx/fullchain.crt;
	ssl_certificate_key /xxx-path-to-cert-xxx/cert.key;
	# SSL 安全配置
	ssl_protocols TLSv1.2 TLSv1.3;
	ssl_prefer_server_ciphers on;
	ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
	ssl_session_timeout 1d;
	ssl_session_cache shared:SSL:10m;
	# 安全头
	add_header X-Frame-Options "SAMEORIGIN";
	add_header X-Content-Type-Options "nosniff";
	add_header X-XSS-Protection "1; mode=block";
	# 根路径与静态文件
	root /xxx-path-to-webroot-xxx/;
	index index.html index.htm;
	location / {
		try_files $uri $uri/ /index.html;
		autoindex off;
	}
	# API 代理
	location /api {
		proxy_pass  http://xxx.xxx.xxx.xxx:yyyy;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded-Proto $scheme;
	}
	# 静态文件缓存
	location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
		try_files $uri =404;
		expires 1y;
		add_header Cache-Control "public, immutable";
	}
	# 日志
	access_log /xxx-path-to-log-xxx/access.log;
	error_log /xxx-path-to-log-xxx/error.log;
}
```
