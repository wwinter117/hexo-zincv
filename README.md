# hexo-zincv

## nginx

### 配置 Nginx 以服务静态网站

创建 Nginx 配置文件： 你可以在 /etc/nginx/sites-available/ 目录下创建一个新的配置文件来配置站点。例如，创建一个名为 hexo-zincv 的配置文件。

```
server {
listen 80;  # 监听80端口，默认HTTP端口
server_name your_domain_or_ip;  # 替换为你的域名或服务器IP

    root /root/dev/github/hexo-zincv/public;  # 静态网站的根目录
    index index.html;  # 默认的首页文件

    location / {
        try_files $uri $uri/ =404;  # 尝试找到文件，如果找不到则返回404
    }

    # 如果使用HTTPS，可以在这里添加SSL证书配置
    # listen 443 ssl;
    # ssl_certificate /path/to/ssl_certificate.crt;
    # ssl_certificate_key /path/to/ssl_certificate.key;
}
```

root：指定网站的根目录，这里是 /root/dev/github/hexo-zincv/public。
server_name：将 your_domain_or_ip 替换为你的网站域名或服务器 IP 地址。

### 启用站点配置

在 sites-available 目录下的站点配置文件需要链接到 sites-enabled 目录，才能让 Nginx 启用它：

sudo ln -s /etc/nginx/sites-available/hexo-zincv /etc/nginx/sites-enabled/

### 测试 Nginx 配置是否正确

在重新启动 Nginx 之前，先检查配置文件是否有语法错误：
sudo nginx -t
如果输出 syntax is ok，说明配置没有问题。

### 重新加载 Nginx, 使新配置生效：

sudo systemctl reload nginx

## 设置防火墙规则（如果有）

确保你的防火墙允许 HTTP 访问。如果使用 ufw，可以通过以下命令开放 HTTP 和 HTTPS 流量：

bash
复制代码
sudo ufw allow 'Nginx Full'

## 访问静态网站

现在，你可以通过浏览器访问你的网站了：

## 可选：启用 HTTPS（SSL/TLS 证书）

为了让你的网站更加安全，推荐启用 HTTPS，使用 SSL/TLS 证书。你可以通过 Let's Encrypt 来获取免费的 SSL 证书。

安装 Certbot（Let's Encrypt 客户端）：
Ubuntu/Debian：
bash
复制代码
sudo apt install certbot python3-certbot-nginx
CentOS：
bash
复制代码
sudo yum install certbot python3-certbot-nginx
生成 SSL 证书：
使用 Certbot 为你的网站生成 SSL 证书：

bash
复制代码
sudo certbot --nginx -d your_domain
配置自动续期：
Certbot 会自动配置续期任务，但你可以通过以下命令手动检查续期功能：

bash
复制代码
sudo certbot renew --dry-run
总结：
安装 Nginx 并启动服务。
配置 Nginx 服务你的静态网站，设置根目录指向 /root/dev/github/hexo-zincv/public。
测试并重启 Nginx 使配置生效。
可选：启用 SSL/TLS 证书以支持 HTTPS。