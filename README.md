# roboshop-shell

testing this roboshop-shell for this
# no need to install below tools - DevOps Practice RHEL9
dnf install unzip -y
dnf install zip -y
dnf install curl -y
dnf install vim -y
dnf install net-tools -y

#frontend code
dnf module disable nginx -y
dnf module enable nginx:1.24 -y
dnf install nginx -y

cp nginx.conf /etc/nginx/nginx.conf
 

rm -rf /usr/share/nginx/html/* 
curl -o /tmp/frontend.zip https://roboshop-artifacts.s3.amazonaws.com/frontend-v3.zip

cd /usr/share/nginx/html 
unzip /tmp/frontend.zip

systemctl restart nginx 
systemctl enable nginx 
systemctl status nginx

# nginx.conf file create in same directory example roboshop-shell(dir) inside the file nginx.conf and frontend 

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log notice;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;

        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }

        location /images/ {
          expires 5s;
          root   /usr/share/nginx/html;
          try_files $uri /images/placeholder.jpg;
        }
        location /api/catalogue/ { proxy_pass http://catalogue.sanatasmia.site:8080; }
        location /api/user/ { proxy_pass http://user.sanatasmia.site:8080; }
        location /api/cart/ { proxy_pass http://cart.sanatasmia.site:8080; }
        location /api/shipping/ { proxy_pass http://shipping.sanatasmia.site:8080; }
        location /api/payment/ { proxy_pass http://payment.sanatasmia.site:8080; }

        location /health {
          stub_status on;
          access_log off;
        }

    }
}

# replace localhost by domain name and keep the port :8080 as it is. example :http://catalogue.sanatasmia.site:8080; exept that no more changes required.
