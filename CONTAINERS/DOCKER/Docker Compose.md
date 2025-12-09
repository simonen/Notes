
PHP + Apache

`docker-compose.yaml`
```
services:
	web:
		image: php:8.0-apache
		volumes:
			- <src>:<dest>
		ports:
			- <published>:<target>
		
```

NGINX + PHP requires two separate services running in conjunction - `nginx` and `php:*-fpm` since NGINX cannot load a `mod_php` unlike Apache

```
services:
	web:     # nginx container
	php-fpm  # php-fpm container
```

```
services:
  web:
    image: nginx:latest
    volumes:
      - ./app:/var/www/html
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    ports:
      - 8080:80
    depends_on:
      - php-fpm
    networks:
      - app-network
        
  php-fpm:
    image: php:8-fpm
    volumes:
      - ./app:/var/www/html
    networks:
      - app-network
        
  db:  
  image: db-image  
  networks:  
    - app-network  
  environment:  
    MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}

networks:
  app-network:   # Network created automatically
```

Environment variables can be defined in a `.env` file.

`nginx.conf`
```
server {
    listen 80;
    root /var/www/html;

    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass php-fpm:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /var/www/html$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

Test `index.php` page

`index.php`
```
<?php
print "<i>Served by: <b>" . gethostname() . "</b></i><br>\n";

print "<pre>";
system("ls -la");
system("pwd");
print "</pre>";
?>
```