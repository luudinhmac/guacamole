Hướng dẫn cài đặt guacamole làm remote gateway server trên ubuntu 18.04
=============
wget https://raw.githubusercontent.com/luudinhmac/guacamole/main/guac-install.sh

chmod +x guac-install.sh

./guac-install.sh


# Cài đặt NGIXN làm Reverse Proxy

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

# Kích hoạt virtual host nginx

ln -s /etc/nginx/sites-available/guacamole.conf /etc/nginx/sites-enabled/

systemctl restart nginx


# Cấu hình SSL cho nginx

Generate SSL/TLS Self-signed Certificate
In this guide, for demonstration purposes, we are going to use self-signed certificates. You can however obtain the trusted CA certificate, otherwise, this will suffice.

```sh
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/guacamole-selfsigned.key -out /etc/ssl/certs/guacamole-selfsigned.crt
```
This will generate the Self-signed key and certificate. When the command runs, you will be prompted to provide some few information. Enter the appropriate information.

```sh
...
Generating a 2048 bit RSA private key
.....................................................................................................+++
.....................+++
writing new private key to '/etc/ssl/private/guacamole-selfsigned.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:VN
State or Province Name (full name) [Some-State]:HCM
Locality Name (eg, city) []:HCM
Organization Name (eg, company) [Internet Widgits Pty Ltd]:CSM
Organizational Unit Name (eg, section) []:IT
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []: 
Configure Nginx to use the Certificates
Once you have the keys in place, proceed to configure Nginx to use the SSL/TLS certificates just generated. In this guide, we will use some of the recommendations on the Cipherli.st.
```

```sh
vim /etc/nginx/sites-available/nginx-guacamole-ssl
server {
	listen 80;
	server_name guacamole.example.com;
	return 301 https://$host$request_uri;
}
server {
	listen 443 ssl;
	server_name guacamole.example.com;

	root /var/www/html;

	index index.html index.htm index.nginx-debian.html;
    
    	ssl_certificate /etc/ssl/certs/guacamole-selfsigned.crt;
	ssl_certificate_key /etc/ssl/private/guacamole-selfsigned.key;

	ssl_protocols TLSv1.2 TLSv1.3;
	ssl_prefer_server_ciphers on; 
	ssl_dhparam /etc/nginx/dhparam.pem;
	ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384;
	ssl_ecdh_curve secp384r1;
	ssl_session_timeout  10m;
	ssl_session_cache shared:SSL:10m;
	resolver 192.168.42.129 8.8.8.8 valid=300s;
	resolver_timeout 5s; 
	add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
	add_header X-Frame-Options DENY;
	add_header X-Content-Type-Options nosniff;
	add_header X-XSS-Protection "1; mode=block";

	access_log  /var/log/nginx/guac_access.log;
	error_log  /var/log/nginx/guac_error.log;

	location / {
		    proxy_pass http://guacamole.example.com:8080/guacamole/;
		    proxy_buffering off;
		    proxy_http_version 1.1;
		    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		    proxy_set_header Upgrade $http_upgrade;
		    proxy_set_header Connection $http_connection;
		    proxy_cookie_path /guacamole/ /;
	}

}
```
Next, generate Deffie-Hellman certificate to ensure a secured key exchange. The -dsaparam option is added to speed up the generation.
```sh
openssl dhparam -dsaparam -out /etc/nginx/dhparam.pem 4096
```
Once that is done, activate Nginx Guacamole configuration.

```sh
ln -s /etc/nginx/sites-available/nginx-guacamole-ssl /etc/nginx/sites-enabled/
```
Verify Nginx configuration.

```sh 
nginx -t
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok

nginx: configuration file /etc/nginx/nginx.conf test is successful
Restart Nginx if all is good.
```sh
systemctl restart nginx
```
