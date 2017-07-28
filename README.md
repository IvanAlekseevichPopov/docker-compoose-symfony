# Docker Symfony (Php-fpm - Nginx - Mariadb - Easticsearch - Redis)

## Installation
1. Create a `.env` from the `.env.dist` file. Adapt it according to your symfony application

    ```bash
    cp .env.dist .env
    ```

2. Build/run containers with (with and without detached mode)

    ```bash
    $ docker-compose build
    $ docker-compose up -d
    ```
3. Prepare Symfony app
    1. Update app/config/parameters.yml

        ```yml
        # path/to/your/symfony-project/app/config/parameters.yml
        parameters:
            database_host: db
        ...
        elastic_clients:
                servers:
                    - { host: elasticsearch, port: 9200 }
        ...
        redis_sessions_dsn: 'redis://redis_6380:6380/2'
        redis_config_master_dsn: 'redis://redis_6390:6390'
        redis_config_slave_dsn: 'redis://redis_6379:6379'
        ```

    2. 
    
4. Move your data to containers
    1. MariaDB (check your oun paths on host machine):
        ``` bash
        cp /source_sql_dump_path/dump.sql ./mariadb/conf/
        docker-compose exec db /bin/bash #Enter to database container
            #This actions inside container
            mysql -pYOUR_ROOT_PASS DATABASE_NAME < /etc/mysql/conf.d/dump.sql
            exit
        rm -f ./mariadb/conf/dump.sql
        ```
        тут проблемы с правами на эластик
    2. Elasticsearch (check your oun paths on host machine):
       ```bash
       cp -r /var/lib/elasticsearch ./data/
       ```
5. Enjoy :-)

## How it works?

Have a look at the `docker-compose.yml` file, here are the `docker-compose` built images:

* `db`: This is the MariaDB database container,
* `php`: This is the PHP-FPM container in which the application volume is mounted,
* `nginx`: This is the Nginx webserver container in which application volume is mounted too,
* `elasticsearch`: This is a elasticsearch 2.4.5 container
* `node`: This is a nodejs container to build React
* `redis_*`: This is redis container
* `phpmyadmin`: This is container with phpMyAdmin panel


This results in the following running containers:

```bash
$ docker-compose ps
            Name                           Command               State                       Ports                     
-----------------------------------------------------------------------------------------------------------------------
dockersymfony_db_1              docker-entrypoint.sh mysqld      Up      0.0.0.0:3306->3306/tcp                        
dockersymfony_elasticsearch_1   /docker-entrypoint.sh elas ...   Up      0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp
dockersymfony_nginx_1           nginx                            Up      0.0.0.0:443->443/tcp, 0.0.0.0:80->80/tcp      
dockersymfony_node_1            npm run start                    Up      0.0.0.0:3000->3000/tcp                        
dockersymfony_php_1             docker-php-entrypoint php-fpm    Up      9000/tcp                                      
dockersymfony_phpmyadmin_1      /run.sh phpmyadmin               Up      0.0.0.0:8080->80/tcp                          
dockersymfony_redis_6379_1      docker-entrypoint.sh redis ...   Up      0.0.0.0:6379->6379/tcp                        
dockersymfony_redis_6380_1      docker-entrypoint.sh redis ...   Up      6379/tcp, 0.0.0.0:6380->6380/tcp              
dockersymfony_redis_6390_1      docker-entrypoint.sh redis ...   Up      6379/tcp, 0.0.0.0:6390->6390/tcp        
```
OR
```bash
[ivan@fedora docker-symfony]$ docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}"
CONTAINER ID        NAMES                           IMAGE                         PORTS                                            STATUS
7c3cc9071f1b        dockersymfony_nginx_1           dockersymfony_nginx           0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp         Up 47 minutes
471f94a60b40        dockersymfony_node_1            dockersymfony_node            0.0.0.0:3000->3000/tcp                           Up 47 minutes
64c90813e300        dockersymfony_redis_6380_1      redis                         6379/tcp, 0.0.0.0:6380->6380/tcp                 Up 47 minutes
c37d6e3b41ec        dockersymfony_db_1              mariadb:10.1                  0.0.0.0:3306->3306/tcp                           Up 47 minutes
421bf4633b2a        dockersymfony_php_1             dockersymfony_php             9000/tcp                                         Up 47 minutes
6734114971ed        dockersymfony_redis_6390_1      redis                         6379/tcp, 0.0.0.0:6390->6390/tcp                 Up 47 minutes
ad5d8b1297fd        dockersymfony_phpmyadmin_1      phpmyadmin/phpmyadmin         0.0.0.0:8080->80/tcp                             Up 47 minutes
531ed2066f6a        dockersymfony_elasticsearch_1   dockersymfony_elasticsearch   0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   Up 47 minutes
0b1b02b4a07a        dockersymfony_redis_6379_1      redis                         0.0.0.0:6379->6379/tcp                           Up 47 minutes
[ivan@fedora docker-symfony]$ 
```


## Useful commands

```bash
# bash commands
$ docker-compose exec php bash

# Composer (e.g. composer update)
$ docker-compose exec php composer update

# SF commands (Tips: there is an alias inside php container)
$ docker-compose exec php php /var/www/symfony/app/console cache:clear # Symfony2
$ docker-compose exec php php /var/www/symfony/bin/console cache:clear # Symfony3
# Same command by using alias
$ docker-compose exec php bash
$ sf cache:clear

# Retrieve an IP Address (here for the nginx container)
$ docker inspect --format '{{ .NetworkSettings.Networks.dockersymfony_default.IPAddress }}' $(docker ps -f name=nginx -q)
$ docker inspect $(docker ps -f name=nginx -q) | grep IPAddress

# MySQL commands
$ docker-compose exec db mysql -uroot -p"root"

# Check CPU consumption
$ docker stats $(docker inspect -f "{{ .Name }}" $(docker ps -q))

# Delete all containers
$ docker rm $(docker ps -aq)

# Delete all images
$ docker rmi $(docker images -q)
```