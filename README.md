# Oracle-Web

nginx配置

```Nginx
    upstream oracle_server{
        server 10.0.0.1:5001; # 服务ip和端口
    }
    server {
        listen       5000 default_server; # 前端端口（端口需要开通策略且不能被占用）
        server_name  127.0.0.1;           # 服务器ip，也可配置为域名
        location / {
                root   /data/Oracle-Web/dist;   # 前端文件路径(文件需要有权限访问)
                index  index.html index.htm;
                try_files $uri $uri/ /index.html =404;
        }

        include /etc/nginx/default.d/*.conf;

        location /oracle {
                    proxy_pass    http://oracle_server/;                    
                    proxy_set_header                Host                         $host;
                    proxy_set_header                X-Real-IP                 $remote_addr;
                    proxy_set_header                X-Forwarded-For         $proxy_add_x_forwarded_for;
        }
    }
```