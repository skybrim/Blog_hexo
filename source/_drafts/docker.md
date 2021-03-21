---
title: docker
comments: true
date: 2021-03-21 17:43:12
tags: [docker, inbox]
---

## docker 示例

dockerfile 示例

```
FROM webdevops/php-apache:8.0-alpine

# 首先配置 vhost
COPY vhost.conf /opt/docker/etc/httpd/vhost.conf
# COPY web.vhost.conf /opt/docker/etc/httpd/vhost.common.d/

# 然后运行初始化脚本
# https://dockerfile.readthedocs.io/en/latest/content/Customization/provisioning.html
COPY init.sh /opt/docker/provision/entrypoint.d/
#CMD chmod +x /opt/docker/provision/entrypoint.d/init.sh
ADD supervisord-proxy.conf /opt/docker/etc/supervisor.d/prism-proxy.conf

EXPOSE 80 
EXPOSE 8802
```

docker-compose.yaml 示例

```
version: '2'
services:
  mariadb:
    image: 'mariadb:10.5.8-focal'
    volumes:
      - 'mariadb_data:/var/lib/mysql'
    environment:
      - MYSQL_ROOT_PASSWORD=theVeryp@ssw0rd
      - MYSQL_DATABASE=prism
    ports:
      - '3306:3306'  
  app:
    #image: 'webdevops/php-apache:8.0-alpine'
    build: './docker/web/'
    ports:
      - '8801:80'
      - '8802:8802'
      - '443:443'
    volumes:
      - './:/app'
    depends_on:
      - mariadb
    environment:
      - DB_HOST=mariadb
      - DB_PORT=3306
      - DB_USERNAME=root
      - DB_DATABASE=prism
      - DB_PASSWORD=theVeryp@ssw0rd
volumes:
  mariadb_data:
```

## docker 命令备忘

### 