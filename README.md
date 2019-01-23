# OrangeHRM Development Environment
[![Docker Automated](https://img.shields.io/docker/automated/orangehrm/orangehrm-environment-images.svg)](https://hub.docker.com/r/orangehrm/orangehrm-environment-images/) [![Docker Status](https://img.shields.io/docker/build/orangehrm/orangehrm-environment-images.svg)](https://hub.docker.com/r/orangehrm/orangehrm-environment-images/) [![Docker Pulls](https://img.shields.io/docker/pulls/orangehrm/orangehrm-environment-images.svg)](https://hub.docker.com/r/orangehrm/orangehrm-environment-images/) [![Build Status](https://travis-ci.org/orangehrm/orangehrm-dev-environment.svg?branch=master)](https://travis-ci.org/orangehrm/orangehrm-dev-environment) [![BCH compliance](https://bettercodehub.com/edge/badge/orangehrm/orangehrm-dev-environment?branch=master)](https://bettercodehub.com/)

## Introduction
orangehrm-dev-environment is a dockerized development environment for OrangeHRM. Usually it will take hours to configure and prepare the development environment for orangehrm system. This project will save the developers time.

This environment will depends on containers of [orangehrm-dev-image](https://hub.docker.com/r/orangehrm/orangehrm-environment-images/),[mysql](https://hub.docker.com/_/mysql/) and [phpmyadmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin/).
## Prerequisites
- Docker engine installed.([Get docker](https://docs.docker.com/engine/installation/))
- Minimum docker version 17.3
- Minimum docker-compose version 1.12. ([Get docker compose](https://docs.docker.com/compose/install/))


## How to use ?
Make sure mentioned prerequisites are there in your host machine.

1. go to release tab and download the latest release.
2. open terminal and go to the cloned directory
3. run the command `docker-compose up -d` - this command will only create three web containers and mysql 5.5 and 5.7 containers only.
4. To start complete development environment run the command `docker-compose -f docker-compose.yml -f docker-compose-all.yml up -d`
5. Moving from complete version to basic version - `docker-compose up -d --remove-orphans`  
6. run `docker ps` and make sure all the containers are up and running.
7. Your web root will be _/ohrm_dev_ directory and make sure to put your project in that folder.

**Note** : [Video Tutorial](https://www.youtube.com/watch?v=fURFe-tARyk)

## Containers

| Container Name | Service Name in docker-compose.yml | Description | IP Address | Used Ports |
|----------------|------------------------------------|-------------|------------|------------|
| dev_web_70     | web70                              | PHP 7.0     | 10.5.0.70  | 9070,3000  |
| dev_web_56     | web56                              | PHP 5.6     | 10.5.0.56  | 9056,3001  |
| dev_web_71     | web71                              | PHP 7.1     | 10.5.0.71  | 9071       |
| dev_mysql_55   | db55                               | MySQL 5.5   | 10.5.0.5   | 3306       |
| dev_mysql_57   | db57                               | MySQL 5.7   | 10.5.0.6   | 3306       |
| dev_mariadb_101| db10                               | MariaDB 10.1| 10.5.0.7   | 3306       |
| dev_phpmyadmin | phpmyadmin                         | phpMyAdmin  | 10.5.0.20  | 9090       |
| dev_openldap   | openldap                           | openldap    | 10.5.0.22  | 700,701    |
|dev_phpldapadmin| phpldapadmin                       | ldapadmin   | 10.5.0.21  | 9093       |
| dev_mailcatcher| mailcatcher                        | mailcatcher | 10.5.0.23  | 1080,1025  |
## Install orangehrm eagle-core inside the container 
1. Get a checkout from svn to ohrm_dev directory.
2. To get the named virtual hosts to work, add the project folder name to /etc/hosts file (`127.0.0.1 folderName`).
3. access from your browser (`https://folderName`). If you have changed the default port configuration in dev_web container then you can access using `htttps://folderName:portNumber`
4. Continue installation by installing system as normal way. (you can have access to inside of dev_web_56 container by running the command `docker exec -it dev_web_56 bash`)

## Default configurations
Developer can override the default configurations if they want by simply adding a docker-compose.override.yml file.It is better to have some knowledge on docker-compose file. ([docker-compose file reference](https://docs.docker.com/compose/compose-file/))
### Default configurations in dev_web_56 container
```
ports:
  - "443:443"
volumes:
  - ./ohrm_dev:/var/www/html
  - ./config/php5/apache2/php.ini:/etc/php5/apache2/php.ini
  - ./config/php5/cli/php.ini:/etc/php5/cli/php.ini
  - ./config/mysql-client:/etc/mysql
  - ./config/apache2/sites-available:/etc/apache2/sites-available
  - ./config/apache2/cert:/etc/apache2/cert
  - ./logs/web_logs/55:/var/log/apache2
  - /etc/localtime:/etc/localtime
  - ./config/xhgui/config.php:/usr/local/src/xhgui/config/config.php
```
#### Example - Overriding port and web root
* Sets apache port to 8080
* Changes web root directory to /home/john/web (from default of ./ohrm_dev)
```
services:
  web56:
    ports:
      - "8080:443"
    volumes:
      - /home/john/web:/var/www/html
```

If you want to change any default apache or php configurations you can find them under _/config_ folder.

### Default configurations in dev_mysql container
```
expose:
  - "3306"
volumes:
  - ./config/mysql-server:/etc/mysql
  - ./logs/mysql_logs:/var/log
  - /etc/localtime:/etc/localtime
  - mysql:/var/lib/mysql
environment:
  MYSQL_ROOT_PASSWORD: 1234
```
You can change exposed port as mentioned in dev_web_55 container configuration. For more information about this container refer into [mysql official image](https://hub.docker.com/_/mysql/).
### Default configurations in dev_phpmyadmin container
```
volumes:
  - /etc/localtime:/etc/localtime
links:
  - db
ports:
  - "9090:80"
environment:
  PMA_HOST: db
```
More about phpmyadmin container can find in docker hub [phpmyadmin image](https://hub.docker.com/r/phpmyadmin/phpmyadmin/)

### Additional information
1. Can access a shell inside of any containers by running `docker exec -it <containerID or ContainerName> /bin/bash  -c "TERM=$TERM; exec bash"`
2. Can restart containers using `docker-compose restart`
3. Can stop containers using `docker-compose stop`
2. Developer can find log files for each container from _/logs_ directory. ( Also possible to get logs of containers by running the command `docker logs <container ID>`)
3. Developer can find configurations for apache, php, mysql-client, mysql-server, etc from _/config directory.
4. If user is using Linux, the docker commands may require sudo permissions. This can be fixed by adding the user to the docker user group.`sudo usermod -aG docker <username>`
5. web 7.0 container doesn't include memcache,ereg and stats PHP modules due to compatibility issues.
6. In web 5.6 container, port 3000 is mapped to 3000 port locally.
7. In web 7.0 container, port 3000 is mapped to 3001 port locally.
8. To catch any emails from the system using mailcatcher set the email configuration as follows.
`SMTP Host : dev_mailcatcher, SMTP Port : 1025, Use SMTP Authentication : No, Use Secure Connection : No`
9. View mailcatcher from http://localhost:1080/
