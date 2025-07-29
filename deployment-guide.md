# Ubuntu服务器部署指南

本指南详细介绍如何将构图大师ComposeAI官方网站部署到Ubuntu服务器上。

## 🚀 部署概览

### 部署架构
```
用户 → 域名 → Cloudflare/CDN → Nginx → 静态文件
```

### 技术栈
- **操作系统**: Ubuntu 20.04/22.04 LTS
- **Web服务器**: Nginx
- **SSL证书**: Let's Encrypt (免费)
- **进程管理**: systemd
- **防火墙**: UFW

## 📋 前置准备

### 1. 服务器要求
- **最低配置**: 1核CPU, 1GB内存, 10GB存储
- **推荐配置**: 2核CPU, 2GB内存, 20GB存储
- **网络**: 公网IP地址
- **系统**: Ubuntu 20.04/22.04 LTS

### 2. 域名准备
- 已购买的域名（如：composeai.app）
- DNS管理权限
- 可选：CDN服务（如Cloudflare）

### 3. 本地工具
- SSH客户端
- SCP/SFTP工具（如FileZilla）
- 或者Git（推荐）

## 🔧 服务器初始配置

### 1. 连接服务器
```bash
# 使用SSH连接服务器
ssh root@your-server-ip
# 或使用用户账户
ssh username@your-server-ip
```

### 2. 更新系统
```bash
# 更新包列表
sudo apt update

# 升级系统
sudo apt upgrade -y

# 安装必要工具
sudo apt install -y curl wget git unzip
```

### 3. 创建部署用户（推荐）
```bash
# 创建专用用户
sudo adduser deploy

# 添加到sudo组
sudo usermod -aG sudo deploy

# 切换到deploy用户
su - deploy
```

### 4. 配置SSH密钥（可选但推荐）
```bash
# 在本地生成SSH密钥对
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"

# 复制公钥到服务器
ssh-copy-id deploy@your-server-ip
```

## 🌐 安装和配置Nginx

### 1. 安装Nginx
```bash
# 安装Nginx
sudo apt install -y nginx

# 启动Nginx
sudo systemctl start nginx

# 设置开机自启
sudo systemctl enable nginx

# 检查状态
sudo systemctl status nginx
```

### 2. 配置防火墙
```bash
# 启用UFW防火墙
sudo ufw enable

# 允许SSH
sudo ufw allow ssh

# 允许HTTP和HTTPS
sudo ufw allow 'Nginx Full'

# 检查状态
sudo ufw status
```

### 3. 测试Nginx
访问 `http://your-server-ip`，应该看到Nginx默认页面。

## 📁 部署网站文件

### 方法1: 使用Git（推荐）

#### 1. 在服务器上克隆代码
```bash
# 创建网站目录
sudo mkdir -p /var/www/composeai
sudo chown -R deploy:deploy /var/www/composeai

# 进入目录
cd /var/www/composeai

# 如果有Git仓库
git clone https://github.com/your-username/composeai-website.git .

# 或者创建Git仓库并推送本地文件
```

#### 2. 设置Git自动部署（可选）
```bash
# 创建部署脚本
cat > /home/deploy/deploy.sh << 'EOF'
#!/bin/bash
cd /var/www/composeai
git pull origin main
sudo systemctl reload nginx
echo "Deployment completed at $(date)"
EOF

# 设置执行权限
chmod +x /home/deploy/deploy.sh
```

### 方法2: 使用SCP/SFTP

#### 1. 从本地上传文件
```bash
# 在本地执行，上传整个网站目录
scp -r /path/to/composeai-website/* deploy@your-server-ip:/var/www/composeai/

# 或使用rsync（推荐）
rsync -avz --delete /path/to/composeai-website/ deploy@your-server-ip:/var/www/composeai/
```

#### 2. 设置文件权限
```bash
# 在服务器上设置权限
sudo chown -R www-data:www-data /var/www/composeai
sudo chmod -R 755 /var/www/composeai
```

## ⚙️ 配置Nginx虚拟主机

### 1. 创建网站配置文件
```bash
sudo nano /etc/nginx/sites-available/composeai
```

### 2. 基础配置（HTTP）
```nginx
server {
    listen 80;
    listen [::]:80;
    
    server_name composeai.app www.composeai.app;
    root /var/www/composeai;
    index index.html;
    
    # 日志文件
    access_log /var/log/nginx/composeai_access.log;
    error_log /var/log/nginx/composeai_error.log;
    
    # 主页面
    location / {
        try_files $uri $uri/ =404;
    }
    
    # 静态文件缓存
    location ~* \.(css|js|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        add_header Vary Accept-Encoding;
    }
    
    # Gzip压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;
    
    # 安全头部
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
}
```

### 3. 启用网站配置
```bash
# 创建软链接
sudo ln -s /etc/nginx/sites-available/composeai /etc/nginx/sites-enabled/

# 删除默认配置（可选）
sudo rm /etc/nginx/sites-enabled/default

# 测试配置
sudo nginx -t

# 重新加载Nginx
sudo systemctl reload nginx
```

## 🔒 配置SSL证书

### 1. 安装Certbot
```bash
# 安装Certbot
sudo apt install -y certbot python3-certbot-nginx
```

### 2. 获取SSL证书
```bash
# 为域名获取证书
sudo certbot --nginx -d composeai.app -d www.composeai.app

# 按提示输入邮箱和同意条款
```

### 3. 自动续期
```bash
# 测试自动续期
sudo certbot renew --dry-run

# 查看定时任务
sudo crontab -l
```

### 4. 完整的HTTPS配置
Certbot会自动更新Nginx配置，最终配置类似：

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name composeai.app www.composeai.app;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    
    server_name composeai.app www.composeai.app;
    root /var/www/composeai;
    index index.html;
    
    # SSL配置
    ssl_certificate /etc/letsencrypt/live/composeai.app/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/composeai.app/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
    
    # 其他配置...
}
```

## 🌍 DNS配置

### 1. A记录配置
在域名DNS管理面板添加：
```
类型: A
名称: @
值: your-server-ip
TTL: 3600

类型: A
名称: www
值: your-server-ip
TTL: 3600
```

### 2. 使用Cloudflare（推荐）

#### 优势
- 免费CDN加速
- DDoS防护
- 额外的安全功能
- 更好的缓存控制

#### 配置步骤
1. 注册Cloudflare账户
2. 添加域名到Cloudflare
3. 更新域名的Nameserver
4. 在Cloudflare添加DNS记录
5. 启用"Proxied"状态

## 🔧 性能优化

### 1. Nginx优化配置
```bash
sudo nano /etc/nginx/nginx.conf
```

```nginx
http {
    # 基础优化
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    
    # 客户端缓冲区
    client_body_buffer_size 128k;
    client_max_body_size 10m;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 4k;
    
    # Gzip压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 10240;
    gzip_proxied expired no-cache no-store private must-revalidate auth;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/javascript
        application/xml+rss
        application/json;
    
    # 包含其他配置文件
    include /etc/nginx/mime.types;
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

### 2. 系统级优化
```bash
# 调整文件描述符限制
echo "* soft nofile 65535" | sudo tee -a /etc/security/limits.conf
echo "* hard nofile 65535" | sudo tee -a /etc/security/limits.conf

# 优化内核参数
sudo nano /etc/sysctl.conf
```

添加以下内容：
```
# 网络优化
net.core.rmem_default = 262144
net.core.rmem_max = 16777216
net.core.wmem_default = 262144
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 65536 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216
```

应用配置：
```bash
sudo sysctl -p
```

## 📊 监控和日志

### 1. 日志管理
```bash
# 查看访问日志
sudo tail -f /var/log/nginx/composeai_access.log

# 查看错误日志
sudo tail -f /var/log/nginx/composeai_error.log

# 配置日志轮转
sudo nano /etc/logrotate.d/nginx
```

### 2. 系统监控
```bash
# 安装htop
sudo apt install -y htop

# 监控系统资源
htop

# 检查磁盘使用
df -h

# 检查内存使用
free -h
```

### 3. 网站监控脚本
```bash
# 创建监控脚本
cat > /home/deploy/monitor.sh << 'EOF'
#!/bin/bash

# 检查网站是否可访问
if curl -f -s https://composeai.app > /dev/null; then
    echo "$(date): Website is UP"
else
    echo "$(date): Website is DOWN" | mail -s "Website Alert" admin@example.com
    # 重启Nginx
    sudo systemctl restart nginx
fi
EOF

chmod +x /home/deploy/monitor.sh

# 添加到定时任务
(crontab -l 2>/dev/null; echo "*/5 * * * * /home/deploy/monitor.sh >> /var/log/website-monitor.log") | crontab -
```

## 🔐 安全加固

### 1. 防火墙配置
```bash
# 只允许必要端口
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

### 2. SSH安全
```bash
# 编辑SSH配置
sudo nano /etc/ssh/sshd_config
```

修改以下配置：
```
# 禁用root登录
PermitRootLogin no

# 使用密钥认证
PasswordAuthentication no
PubkeyAuthentication yes

# 更改默认端口（可选）
Port 2222

# 限制登录尝试
MaxAuthTries 3
```

重启SSH服务：
```bash
sudo systemctl restart ssh
```

### 3. 安装Fail2ban
```bash
# 安装Fail2ban
sudo apt install -y fail2ban

# 配置Fail2ban
sudo nano /etc/fail2ban/jail.local
```

添加配置：
```ini
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3

[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log

[nginx-http-auth]
enabled = true
port = http,https
logpath = /var/log/nginx/error.log
```

启动服务：
```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

## 🚀 自动化部署

### 1. 创建部署脚本
```bash
cat > /home/deploy/auto-deploy.sh << 'EOF'
#!/bin/bash

set -e

echo "Starting deployment..."

# 进入网站目录
cd /var/www/composeai

# 备份当前版本
cp -r . ../composeai-backup-$(date +%Y%m%d-%H%M%S)

# 拉取最新代码
git pull origin main

# 设置权限
sudo chown -R www-data:www-data .
sudo chmod -R 755 .

# 测试Nginx配置
sudo nginx -t

# 重新加载Nginx
sudo systemctl reload nginx

echo "Deployment completed successfully!"
EOF

chmod +x /home/deploy/auto-deploy.sh
```

### 2. GitHub Actions自动部署（可选）
在GitHub仓库创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy to Server

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: Deploy to server
      uses: appleboy/ssh-action@v0.1.5
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.PRIVATE_KEY }}
        script: |
          cd /var/www/composeai
          git pull origin main
          sudo systemctl reload nginx
```

## 📋 部署检查清单

### 部署前检查
- [ ] 服务器已准备就绪
- [ ] 域名DNS已配置
- [ ] 网站文件已准备
- [ ] SSL证书计划已确定

### 部署过程检查
- [ ] Nginx安装成功
- [ ] 网站文件上传完成
- [ ] Nginx配置正确
- [ ] SSL证书配置成功
- [ ] 防火墙配置正确

### 部署后检查
- [ ] 网站可正常访问
- [ ] HTTPS重定向工作
- [ ] 所有页面加载正常
- [ ] 移动端显示正常
- [ ] 语言切换功能正常
- [ ] 性能测试通过

## 🔧 故障排除

### 常见问题

#### 1. 网站无法访问
```bash
# 检查Nginx状态
sudo systemctl status nginx

# 检查配置文件
sudo nginx -t

# 查看错误日志
sudo tail -f /var/log/nginx/error.log
```

#### 2. SSL证书问题
```bash
# 检查证书状态
sudo certbot certificates

# 手动续期
sudo certbot renew

# 重新获取证书
sudo certbot --nginx -d composeai.app
```

#### 3. 权限问题
```bash
# 重新设置权限
sudo chown -R www-data:www-data /var/www/composeai
sudo chmod -R 755 /var/www/composeai
```

### 性能问题诊断
```bash
# 检查服务器负载
top
htop

# 检查磁盘IO
iostat -x 1

# 检查网络连接
netstat -tuln

# 测试网站响应时间
curl -w "@curl-format.txt" -o /dev/null -s https://composeai.app
```

## 📞 技术支持

如遇到部署问题，可以：

1. 查看相关日志文件
2. 检查系统资源使用情况
3. 验证配置文件语法
4. 联系技术支持：support@composeai.app

---

**部署完成后，您的构图大师ComposeAI官方网站将在互联网上稳定运行！**

记住定期备份数据、更新系统和监控网站状态。

# 同步当前目录到远程服务器（会删除远程多余文件）
rsync -avz --delete ./ yy@composeai.net:/var/www/composeai/

# 或者不删除远程多余文件
rsync -avz ./ yy@composeai.net:/var/www/composeai/