# Php / nginx / mysql docker example


## Tutorial

### Server

Create a project folder on your computer and a docker-compose.yml file with the following contents:

```bash
version: '2'

services:
```

where version defines the current docker-compose version and a services section where our containers will go.

First we add the nginx server under services to serve our files to a host:

```bash
  nginx:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./src:/src
```

Where ports will map the host port to our container port and volumes will map the project files into the container.

We also need to tell the nginx server that our files will live within the src/ folder so we pass in a config file `site.conf` with the following lines

```bash
server {
    index index.html;
    root /src;
}
```

Place an index.html into the src/ folder and run `docker-composer up` and visit [http://localhost:8080](http://localhost:8080)

We should see the contents of our index.html meaning the server works.

### PHP

To be able to run php files on our server we would normally install it to our machine but with docker we can create it as a separate service and link it to nginx.

Let's add the second service in the docker-compose file:

```bash
  php:
    image: php:7-fpm
    volumes:
      - ./src:/src
```

which will also take the src/ folder as an input.

We also need to tell nginx where the php processor is by extending the site.conf file

```bash
server {
    index index.php;
    root /src;

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

Notice we changed the index to `index.php`. We also added a location section to handle all .php file extensions.

To read more about PHP-FPM configs within nginx, read the [docs](https://www.nginx.com/resources/wiki/start/topics/examples/phpfcgi/).

Rename index.html to index.php and try to echo the phpinfo:

```php
<?php
phpinfo();
```

Now if we `docker-compose up` again and visit localhost we should see the information of our current php version.

As a last step we add mysql to the services:

```bash
  mysql:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_ALLOW_EMPTY_PASSWORD: 'yes'
```

After `docker-compose up` again we should be able to connect to the local mysql server on `127.0.0.1` with username `root` and password none.