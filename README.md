# Docker Symfony (Php-fpm - Nginx - Mariadb - Easticsearch - Redis - Nodejs)

### Docker installation(If you have not yet installed docker-compose)
1. Install docker and docker-compose packages for your OS.
    1. Ubuntu/Debian:

    ```bash
       sudo apt-get install docker docker-compose nmap
    ```
    2. Fedora/Centos:

    ```bash
       sudo dnf install docker docker-compose
    ```

2. For Ubuntu/Debian you need to fill /etc/resolv.conf with actual DNS servers
(it needs for correct work of docker internal network)

    ```bash
       sudo bash -c 'echo "nameserver REPLACE_WITH_YOUR_DNS_SERVER_IP" >> /etc/resolv.conf'
    ```
    
3. Add your user to docker group for access to docker commands without sudo:

    ```bash
       sudo groupadd docker
       sudo usermod -aG docker $USER
    ```

4. Enable docker service:

    ```bash
       sudo systemctl enable docker
       sudo systemctl start docker
    ```
    
5. Log out and log back in so that your group membership is re-evaluated.

### Project installation
1. Clone this repository and change folder to project root: 

    ```bash
    git clone git@github.com:IvanAlekseevichPopov/docker-compoose-symfony.git
    cd docker-compoose-symfony
    ```

2. Check, that ports, used in project are not busy:
    1. Check with nmap (or use telnet):
        ```bash
        sudo nmap -sS -O -p80,443,8080,6379,6380,6390,9200,9300,3306,3000 localhost | grep open
        80/tcp   open  http
        443/tcp  open  https
        3000/tcp open  ppp
        3306/tcp open  mysql
        6379/tcp open  redis
        6380/tcp open  unknown
        6390/tcp open  metaedit-ws
        8080/tcp open  http-proxy
        9200/tcp open  wap-wsp
        9300/tcp open  vrace
        #If one of this ports is open - disable corresponding service
        #on port 3000 - close running npm server
        ``` 
    2. Disable services, if necessary:
        1. Ubuntu/Debian:
        ```bash
        #To stop
        systemctl stop elasticsearch redis_6379 redis_6380 redis_6390 nginx mysql
        #To disable autoload
        systemctl disable elasticsearch redis_6379 redis_6380 redis_6390 nginx mysq
        ```
        2. Fedora/Centos:
        ```bash
        #To stop
        systemctl stop elasticsearch redis_6379 redis_6380 redis_6390 nginx mariadb
        #To disable autoload
        systemctl disable elasticsearch redis_6379 redis_6380 redis_6390 nginx mariadb
        ```

1. Create a `.env` from the `.env.dist` file:

    ```bash
    cp .env.dist .env
    ```
    DO NOT FORGOT to adapt it according to your symfony application
    ```bash
    nano .env
    #nano is peace of shit, use vim )
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
            database_port: 3306
            database_name: dionis
            database_user: dionis_user
            database_password: pass
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
    1. Elasticsearch (check your oun paths on host machine):
       ```bash
       rm -rf ./data/elasticsearch; cp -r /var/lib/elasticsearch ./data/
       ```
    2. MariaDB (check your oun paths on host machine):
        ``` bash
        cp /source_sql_dump_path/YOUR_MYSQL_DUMP_NAME.sql ./mariadb/conf/
        docker-compose exec db /bin/bash #Enter to database container
            #This actions inside container
            mysql -pYOUR_ROOT_PASS DATABASE_NAME < /etc/mysql/conf.d/YOUR_MYSQL_DUMP_NAME.sql
            exit
        rm -f ./mariadb/conf/dump.sql
        ```
    3. If you need, make migrations:
        ``` bash
        docker-compose exec php app/console d:m:m
        ```
5. Enjoy :-)

## How it works?

Have a look at the `docker-compose.yml` file, here are the `docker-compose` built images:

* `db`: This is the MariaDB database container,
* `php`: This is the PHP-FPM container in which the application volume is mounted,
* `nginx`: This is the Nginx webserver container in which application volume is mounted too,
* `elasticsearch`: This is a elasticsearch 2.4.5 container
* `node`: This is a nodejs container to build React
* `redis_*`: This is redis containers
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
$ docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}"

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