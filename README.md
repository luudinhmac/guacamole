Hướng dẫn cài đặt guacamole làm remote server
=============
wget https://raw.githubusercontent.com/luudinhmac/guacamole/main/guac-install.sh

chmod +x guac-install.sh

./guac-install.sh


#Cài đặt NGIXN làm Reverse Proxy

apt-get install nginx -y

nano /etc/nginx/sites-available/guacamole.conf

Thêm vào các dòng sau:

server {

listen 80;

server_name your-server-ip;

access_log /var/log/nginx/guac_access.log;

error_log /var/log/nginx/guac_error.log;

location / {

proxy_pass http://your-server-ip:8080/guacamole/;

proxy_buffering off;

proxy_http_version 1.1;

proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

proxy_set_header Upgrade $http_upgrade;

proxy_set_header Connection $http_connection;

proxy_cookie_path /guacamole/ /;

}

}

Kích hoạt virtual host nginx
=============

ln -s /etc/nginx/sites-available/guacamole.conf /etc/nginx/sites-enabled/

systemctl restart nginx
