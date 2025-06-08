# MD-Updater 服务器配置指南

## 概述

本指南详细介绍如何配置服务器环境以支持 MD-Updater 模组的更新功能。服务器需要提供模组列表文件和模组下载服务。

## 服务器要求

### 基础要求

- Web服务器（推荐 Nginx 或 Apache）
- 支持静态文件服务
- 支持HTTP/HTTPS协议
- 足够的存储空间和带宽

### 推荐配置

- 操作系统：Ubuntu 20.04+ 或 CentOS 7+
- Web服务器：Nginx 1.18+
- SSL证书：Let's Encrypt 或商业证书
- 带宽：根据用户数量和模组大小确定

## Nginx 配置

### 1. 基础配置

创建站点配置文件 `/etc/nginx/sites-available/md-updater`：

```nginx
server {
    listen 80;
    server_name your-domain.com;
    
    # 重定向到HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;
    
    # SSL配置
    ssl_certificate /path/to/your/certificate.crt;
    ssl_certificate_key /path/to/your/private.key;
    
    # 网站根目录
    root /var/www/md-updater;
    index index.html;
    
    # 模组文件目录
    location /mods/ {
        alias /var/www/md-updater/mods/;
        autoindex off;
        
        # 设置正确的MIME类型
        location ~* \.jar$ {
            add_header Content-Type application/java-archive;
            add_header Content-Disposition attachment;
        }
    }
    
    # mod_list.json文件
    location /mod_list.json {
        add_header Content-Type application/json;
        add_header Access-Control-Allow-Origin *;
        add_header Cache-Control "no-cache, no-store, must-revalidate";
    }
    
    # 安全配置
    location ~ /\. {
        deny all;
    }
    
    # 日志配置
    access_log /var/log/nginx/md-updater-access.log;
    error_log /var/log/nginx/md-updater-error.log;
}
```

### 2. 启用配置

```bash
# 创建符号链接
sudo ln -s /etc/nginx/sites-available/md-updater /etc/nginx/sites-enabled/

# 测试配置
sudo nginx -t

# 重载配置
sudo systemctl reload nginx
```

## 目录结构

### 推荐的服务器目录结构

```
/var/www/md-updater/
├── mod_list.json          # 模组列表文件
├── mods/                  # 模组文件目录
│   ├── coolmod-1.0.jar
│   ├── awesomemod-2.1.jar
│   └── utilitymod-1.5.jar
├── index.html             # 可选的说明页面
└── logs/                  # 可选的日志目录
```

### 创建目录

```bash
# 创建目录结构
sudo mkdir -p /var/www/md-updater/mods
sudo mkdir -p /var/www/md-updater/logs

# 设置权限
sudo chown -R www-data:www-data /var/www/md-updater
sudo chmod -R 755 /var/www/md-updater
```

## mod_list.json 配置

### 文件格式

`mod_list.json` 文件必须遵循以下JSON格式：

```json
{
  "mods": [
    {
      "name": "coolmod-1.0.jar",
      "url": "https://your-domain.com/mods/coolmod-1.0.jar"
    },
    {
      "name": "awesomemod-2.1.jar", 
      "url": "https://your-domain.com/mods/awesomemod-2.1.jar"
    },
    {
      "name": "utilitymod-1.5.jar",
      "url": "https://your-domain.com/mods/utilitymod-1.5.jar"
    }
  ]
}
```

### 字段说明

- **name**: 模组文件的完整文件名（包括.jar扩展名）
- **url**: 模组文件的完整下载URL

### 重要注意事项

1. **文件名匹配**：`name` 字段必须与实际文件名完全一致
2. **URL有效性**：确保所有URL都可以正常访问
3. **JSON格式**：确保JSON格式正确，注意逗号和引号
4. **编码格式**：使用UTF-8编码保存文件

## 模组文件管理

### 上传模组文件

```bash
# 上传模组文件到服务器
scp local-mod.jar user@server:/var/www/md-updater/mods/

# 设置正确的权限
sudo chown www-data:www-data /var/www/md-updater/mods/local-mod.jar
sudo chmod 644 /var/www/md-updater/mods/local-mod.jar
```

### 更新模组列表

每次添加或删除模组文件后，需要更新 `mod_list.json`：

```bash
# 编辑模组列表
sudo nano /var/www/md-updater/mod_list.json

# 验证JSON格式
python3 -m json.tool /var/www/md-updater/mod_list.json
```

### 自动化脚本

创建自动更新脚本 `/usr/local/bin/update-mod-list.sh`：

```bash
#!/bin/bash

MOD_DIR="/var/www/md-updater/mods"
JSON_FILE="/var/www/md-updater/mod_list.json"
DOMAIN="https://your-domain.com"

# 生成模组列表
echo '{"mods":[' > "$JSON_FILE"

first=true
for file in "$MOD_DIR"/*.jar; do
    if [ -f "$file" ]; then
        filename=$(basename "$file")
        if [ "$first" = true ]; then
            first=false
        else
            echo ',' >> "$JSON_FILE"
        fi
        echo "  {\"name\":\"$filename\",\"url\":\"$DOMAIN/mods/$filename\"}" >> "$JSON_FILE"
    fi
done

echo ']}' >> "$JSON_FILE"

# 设置权限
chown www-data:www-data "$JSON_FILE"
chmod 644 "$JSON_FILE"

echo "模组列表已更新"
```

使脚本可执行：

```bash
sudo chmod +x /usr/local/bin/update-mod-list.sh
```

## 安全配置

### 1. 防火墙配置

```bash
# 允许HTTP和HTTPS流量
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# 启用防火墙
sudo ufw enable
```

### 2. SSL证书配置

使用 Let's Encrypt 获取免费SSL证书：

```bash
# 安装certbot
sudo apt install certbot python3-certbot-nginx

# 获取证书
sudo certbot --nginx -d your-domain.com

# 设置自动续期
sudo crontab -e
# 添加以下行：
# 0 12 * * * /usr/bin/certbot renew --quiet
```

### 3. 访问控制

限制对管理文件的访问：

```nginx
# 在server块中添加
location ~ ^/(logs|scripts|config)/ {
    deny all;
    return 404;
}

# 限制文件上传大小
client_max_body_size 100M;
```

## 监控和日志

### 1. 访问日志分析

```bash
# 查看访问日志
sudo tail -f /var/log/nginx/md-updater-access.log

# 分析下载统计
sudo grep "GET /mods/" /var/log/nginx/md-updater-access.log | wc -l
```

### 2. 错误监控

```bash
# 查看错误日志
sudo tail -f /var/log/nginx/md-updater-error.log

# 检查404错误
sudo grep "404" /var/log/nginx/md-updater-access.log
```

### 3. 磁盘空间监控

```bash
# 检查磁盘使用情况
df -h /var/www/md-updater

# 设置磁盘空间警告
echo "if [ \$(df /var/www/md-updater | tail -1 | awk '{print \$5}' | sed 's/%//') -gt 80 ]; then echo 'Warning: Disk usage > 80%'; fi" | sudo tee /etc/cron.daily/disk-check
```

## 备份策略

### 1. 自动备份脚本

创建备份脚本 `/usr/local/bin/backup-mods.sh`：

```bash
#!/bin/bash

BACKUP_DIR="/backup/md-updater"
SOURCE_DIR="/var/www/md-updater"
DATE=$(date +%Y%m%d_%H%M%S)

# 创建备份目录
mkdir -p "$BACKUP_DIR"

# 备份文件
tar -czf "$BACKUP_DIR/md-updater-$DATE.tar.gz" -C "$SOURCE_DIR" .

# 保留最近7天的备份
find "$BACKUP_DIR" -name "md-updater-*.tar.gz" -mtime +7 -delete

echo "备份完成: md-updater-$DATE.tar.gz"
```

### 2. 定时备份

```bash
# 添加到crontab
sudo crontab -e
# 添加以下行（每天凌晨2点备份）：
# 0 2 * * * /usr/local/bin/backup-mods.sh
```

## 性能优化

### 1. Nginx优化

```nginx
# 在http块中添加
gzip on;
gzip_vary on;
gzip_min_length 1024;
gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

# 缓存配置
location ~* \.(jar)$ {
    expires 1d;
    add_header Cache-Control "public, immutable";
}
```

### 2. 系统优化

```bash
# 增加文件描述符限制
echo "* soft nofile 65536" | sudo tee -a /etc/security/limits.conf
echo "* hard nofile 65536" | sudo tee -a /etc/security/limits.conf

# 优化网络参数
echo "net.core.somaxconn = 65536" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## 故障排除

### 常见问题

**Q: 客户端无法下载mod_list.json**
A: 检查Nginx配置、防火墙设置和DNS解析

**Q: 模组文件下载失败**
A: 检查文件权限、URL正确性和服务器磁盘空间

**Q: JSON格式错误**
A: 使用JSON验证工具检查格式，注意逗号和引号

### 调试命令

```bash
# 测试服务器响应
curl -I https://your-domain.com/mod_list.json

# 测试模组下载
wget https://your-domain.com/mods/example.jar

# 检查Nginx状态
sudo systemctl status nginx

# 验证JSON格式
python3 -m json.tool /var/www/md-updater/mod_list.json
```

## 维护建议

1. **定期更新**：及时更新模组文件和列表
2. **监控日志**：定期检查访问和错误日志
3. **备份数据**：建立可靠的备份机制
4. **安全更新**：及时更新服务器软件
5. **性能监控**：监控服务器性能和带宽使用

---

*本文档最后更新：2025年6月8日*

