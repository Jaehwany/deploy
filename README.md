# Infra
<br>

### Deploy

<br>

1. Docker, Docker-compose 설치

```
$ cd Infra/docker
$ make run
```
<br>

2. Jenkins 설치

```
$ cd Infra/jenkins
$ make run
$ docker logs jenkins
$ sudo chmod +w ${error_volume}
```
<br>

3. certbot 인증서 생성 + nginx 설치

```
$ cd Infra/nginx
$ chmod +x init-letsencrypt.sh
$ sh init-letsencrypt.sh
```
```
$ docker-compose up --build -d
```
<br>

4. nginx https 설정(app.conf 수정)
```
$ sudo vi data/nginx/conf.d/app.conf
```

```
server {
     listen 80;
     listen [::]:80;

     server_name 서버 도메인; 

     location /.well-known/acme-challenge/ {
             allow all;
             root /var/www/certbot;
     }
     
     location / {
         return 301 https://$server_name$request_uri;
     }
}


server {
    listen 443 ssl;
    server_name 서버 도메인;
    server_tokens off;

    ssl_certificate /etc/letsencrypt/live/{서버 도메인}/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/{서버 도메인}/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
	
    include /etc/nginx/conf.d/service-url.inc;
    include /etc/nginx/conf.d/service-url2.inc;
    
    location / {
        proxy_pass $service_port2;
        proxy_set_header    Host                $http_host;
        proxy_set_header    X-Real-IP           $remote_addr;
        proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
    }

    location /api {
        proxy_pass $service_port;
        proxy_set_header    Host                $http_host;
        proxy_set_header    X-Real-IP           $remote_addr;
        proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
    }
}
