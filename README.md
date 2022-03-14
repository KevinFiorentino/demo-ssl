## Docker + NGINX + Certibot SSL

- Install Certibot
    - `apt update`
    - `apt install python-certbot-nginx`

- Levantar NGINX con la configuración para la verificación de Let's Encrypt
- `docker-compose up -d`

- `certbot certonly --webroot -d example.com -d www.example.com`
    - Input the webroot for example.com: (Enter 'c' to cancel): /root/app/demo-ssl/wwwroot        // Apuntar a la raíz de la app

- Se creará en /etc/letsencrypt/live/example.com
- Cambiar configuración NGINX por la necesaria para levantar el proxy en 443


```
# Configuración para verificación de Let's Encrypt
server {
    listen   80;
    server_name   example.com www.example.com;
 
    location ^~ /.well-known/acme-challenge/ {
       default_type "text/plain";
       root     /var/www;
    }
    location = /.well-known/acme-challenge/ {
       return 404;
    }
    location / {
        root   /var/www;
        index  index.html index.htm;
    }
}
```

```
# Configuración del servidor
server {
    listen   443 ssl;
    server_name  example.com;
    ssl_certificate        /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key    /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        root /var/www;
        index index.jsp index.html index.htm index.php;
    }
    location /proxy/ {
        root /var/www;
        index index.jsp index.html index.htm index.php;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;

        proxy_pass http://example:3000/;
    }
}
 
server {
    listen   80;
    server_name   example.com www.example.com;
    return 301 https://$server_name$request_uri;            
}
```