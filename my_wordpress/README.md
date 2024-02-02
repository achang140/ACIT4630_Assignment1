# ACIT 4630 Assignment 1 - Keep Secrets Secret!

* Reference: \
https://github.com/docker/awesome-compose/tree/master/official-documentation-samples/wordpress/

## WordPress Setup 

### Unsafe Way: Secrets Not Kept Privately

CMD / PowerShell: 

1. Create a new project directory and change into the directory

```sh
mkdir my_wordpress && cd my_wordpress 
```

2. Create a `docker-compose.yml` file that launches containers for WordPress and MySQL, with the MySQL instance configured to use volume mounts for data persistence 

* Note: Secrets are exposed 

```YAML
services:
  db:
    # We use a mariadb image which supports both amd64 & arm64 architecture
    image: mariadb:10.6.4-focal
    # If you really want to use MySQL, uncomment the following line
    #image: mysql:8.0.27
    command: '--default-authentication-plugin=mysql_native_password'
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=somewordpress
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=wordpress
      - MYSQL_PASSWORD=wordpress
    expose:
      - 3306
      - 33060
  wordpress:
    image: wordpress:latest
    volumes:
      - wp_data:/var/www/html
    ports:
      - 80:80
    restart: always
    environment:
      - WORDPRESS_DB_HOST=db
      - WORDPRESS_DB_USER=wordpress
      - WORDPRESS_DB_PASSWORD=wordpress
      - WORDPRESS_DB_NAME=wordpress
volumes:
  db_data:
  wp_data:
```

3. Run `docker-compose up -d` 
![docker-compose up Command](/my_wordpress/Images/docker-compose_up_-d_Command.png)

4. Verify 2 containers are created successfully and running 
![docker-compose up Command](/my_wordpress/Images/docker_ps_Command.png)

5. Go to: http://localhost:80/, you should see: 
![WordPress_1](/my_wordpress/Images/WordPress_1.png)
![WordPress_2](/my_wordpress/Images/WordPress_2.png)

6. Bash into the running WordPress container and print off the environment variables: 

```sh
docker exec -it CONTAINER_ID bash 
printenv 
```
![printenv_Command](/my_wordpress/Images/printenv_Command.png)

7. Shutdown and cleanup 

```sh
docker-compose down  # Removes containers and default network, but preserves WordPress database (DB)

docker-compose down --volumes # Removes containers, default network, and the WordPress DB 
```
![docker-compose_down--volume_Command](/my_wordpress/Images/docker-compose_down--volume_Command.png)

### Safe Way: Use .env file with docker-compose.yml 

* Create a new project directory containing a `.env` file and a modified version of `docker-compose.yml`

* Proceed with the identical procedure outlined previously

`.env` File: 
```sh
MYSQL_ROOT_PASSWORD=somewordpress
MYSQL_DATABASE=wordpress
MYSQL_USER=wordpress
MYSQL_PASSWORD=wordpress
WORDPRESS_DB_HOST=db
WORDPRESS_DB_USER=wordpress
WORDPRESS_DB_PASSWORD=wordpress
WORDPRESS_DB_NAME=wordpress
```

`docker-compose.yml` File:
```YAML
services:
  db:
    # We use a mariadb image which supports both amd64 & arm64 architecture
    image: mariadb:10.6.4-focal
    # If you really want to use MySQL, uncomment the following line
    #image: mysql:8.0.27
    command: '--default-authentication-plugin=mysql_native_password'
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    expose:
      - 3306
      - 33060
  wordpress:
    image: wordpress:latest
    volumes:
      - wp_data:/var/www/html
    ports:
      - 80:80
    restart: always
    environment:
      - WORDPRESS_DB_HOST=${WORDPRESS_DB_HOST}
      - WORDPRESS_DB_USER=${WORDPRESS_DB_USER}
      - WORDPRESS_DB_PASSWORD=${WORDPRESS_DB_PASSWORD}
      - WORDPRESS_DB_NAME=${WORDPRESS_DB_NAME}
volumes:
  db_data:
  wp_data:
```
