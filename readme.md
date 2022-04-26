How to install nginx from source

#1. Create folder
```
mkdir -p /opt/nginx 
mkdir /opt/nginx/logs/
mkdir /run
```
#2. Install package require 
```
Centos: yum install pcre pcre-devel zlib zlib-devel openssl openssl-devel gcc -y
Ubuntu: apt-get install libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev openssl
```
#3. Compile
```
./configure  --sbin-path=/usr/bin/nginx --prefix=/opt/nginx --conf-path=/opt/nginx/nginx.conf --error-log-path=/opt/nginx/logs/error.log --http-log-path=/opt/nginx/logs/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module --with-http_realip_module
make
make install
ln -s /opt/nginx/sbin/nginx /sbin/nginx
```

#4. Sample config with ssl:
```
server {
        server_name _;
        client_max_body_size 20M;
        listen 443;
        ssl on;
        #ssl_session_timeout 5m;
        #ssl_protocols SSLv2 SSLv3 TLSv1;
        ssl_certificate /opt/cert/ca.crt;
        ssl_certificate_key /opt/cert/ca.key ;
        ssl_session_cache shared:SSL:10m;
        root /opt/nginx/html;
}

```
#5. Systemctl For Nginx
```
#vim /lib/systemd/system/nginx.service
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/bin/nginx -t
ExecStart=/usr/bin/nginx
ExecReload=/usr/bin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

# Open SSL generate:

#1. Create Private key + CSR
```
# Create private key with pass phase
openssl genrsa -des3 -out ca.key 2048

# Create csr from private key, send csr to Authorize Company
openssl req -new -sha256 -key ca.key -out ca.csr

# Remove pass phase if require
openssl rsa -in ca.key -out ca_nopass.key
```

#2. Create self certificate
```
openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt
```

