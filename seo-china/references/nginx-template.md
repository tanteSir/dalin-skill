# Nginx 配置模板

## 域名统一 + gzip + 缓存 + 安全头

```nginx
# HTTP → HTTPS + 统一到主域名
server {
    listen 80;
    server_name example.cn www.example.cn;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://www.example.cn$request_uri;
    }
}

# 非 www → www（或反过来）
server {
    listen 443 ssl;
    server_name example.cn;

    ssl_certificate /etc/letsencrypt/live/example.cn/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.cn/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;

    return 301 https://www.example.cn$request_uri;
}

# 主 server 块
server {
    listen 443 ssl;
    server_name www.example.cn;

    ssl_certificate /etc/letsencrypt/live/example.cn/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.cn/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;

    # 安全头
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;

    # Gzip 压缩
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript image/svg+xml;
    gzip_min_length 1000;
    gzip_vary on;

    # Docker 内部 DNS
    resolver 127.0.0.11 valid=10s ipv6=off;

    set $backend_upstream http://backend:8000;
    set $frontend_upstream http://dashboard:3000;

    # API 代理
    location /api/ {
        proxy_pass $backend_upstream;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
        proxy_cache off;
        proxy_read_timeout 300s;
    }

    # 静态资源长缓存
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot|webp|mp4|webm)$ {
        proxy_pass $frontend_upstream;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # 前端代理
    location / {
        proxy_pass $frontend_upstream;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
