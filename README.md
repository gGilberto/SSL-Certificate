Generar certificados 
//Para autoridad certificacora
	Genaramos la clave privada
-openssl genrsa -out /etc/ssl/private/cakey.key 2048 -des3
	Genaramos el certificados
-openssl req -new -x509 -keyout /etc/ssl/private/cakey.key -out /etc/ssl/certs/cacert.pem  -days 365

creamos el certificado de cliente a firmar
		Creamos la Key
-openssl genrsa -out /etc/ssl/private/taqueritos.key 2048
	Creamos el SigninRequest
-openssl req -new -key /etc/ssl/private/taqueritos.key -out /etc/ssl/certs/taqueritos.csr
	Firmamos el certificado con nuestra CA
-openssl x509 -req -in /etc/ssl/certs/taqueritos.csr -CA /etc/ssl/certs/cacert.pem -CAkey /etc/ssl/private/cakey.key -CAcreateserial -out /etc/ssl/certs/taqueritos.crt -days 365
--sudo openssl dhparam -out /etc/nginx/dhparam.pem 4096
		CONFIGURAR NGINX
--nano /etc/nginx/snippets/taqueritos.conf
	ssl_certificate /etc/ssl/certs/taqueritos.crt;
	ssl_certificate_key /etc/ssl/private/taqueritos.key;
--nano /etc/nginx/snippets/ssl-params.conf
	
ssl_protocols TLSv1.3;
ssl_prefer_server_ciphers on;
ssl_dhparam /etc/nginx/dhparam.pem; 
ssl_ciphers EECDH+AESGCM:EDH+AESGCM;
ssl_ecdh_curve secp384r1;
ssl_session_timeout  10m;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
# Disable strict transport security for now. You can uncomment the following
# line if you understand the implications.
#add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
--nano /etc/nginx/sites-available/taqueritos
	server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    include snippets/taqueritos.conf;
    include snippets/ssl-params.conf;

root /var/www/taqueritos/html;
        index index.html index.htm index.nginx-debian.html;

  server_name taqueritos.net www.taqueritos.net;

  location / {
                try_files $uri $uri/ =404;
        }

}

server {
    listen 80;
    listen [::]:80;

    server_name taqueritos.net www.taqueritos.net;

    return 302 https://$server_name$request_uri;
}
ufw allow 'Nginx Full'
ufw delete allow 'Nginx HTTP'
