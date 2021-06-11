## docker-rap2部署

```javascript
mkidr -p /data/rap
```

## 启动

```javascript
vi /data/rap/docker-compose.yml
# mail@dongguochao.com
# llitfkitfk@gmail.com
# chibing.fy@alibaba-inc.com

version: "3"

services:
  # frontend
  dolores:
    image: rapteam/rap2-dolores:latest
    ports:
      #冒号前可以自定义前端端口号，冒号后不要动
      - 3000:38081

  # backend
  delos:
    image: rapteam/rap2-delos:latest
    ports:
      # 这里的配置不要改哦
      - 38080:38080
    environment:
      - SERVE_PORT=38080
      # if you have your own mysql, config it here, and disable the 'mysql' config blow
      - MYSQL_URL=mysql # links will maintain /etc/hosts, just use 'container_name'
      - MYSQL_PORT=3306
      - MYSQL_USERNAME=root
      - MYSQL_PASSWD=
      - MYSQL_SCHEMA=rap2

      # redis config
      - REDIS_URL=redis
      - REDIS_PORT=6379

      # production / development
      - NODE_ENV=production
    ###### 'sleep 30 && node scripts/init' will drop the tables
    ###### RUN ONLY ONCE THEN REMOVE 'sleep 30 && node scripts/init'
    command: /bin/sh -c 'node dispatch.js'
    # init the databases
    # command: sleep 30 && node scripts/init && node dispatch.js
    # without init
    # command: node dispatch.js
    depends_on:
      - redis
      - mysql

  redis:
    image: redis:4

  # disable this if you have your own mysql
  mysql:
    image: mysql:5.7
    # expose 33306 to client (navicat)
    ports:
      - 33306:3306
    volumes:
      # change './docker/mysql/volume' to your own path
      # WARNING: without this line, your data will be lost.
      - "./docker/mysql/volume:/var/lib/mysql"
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --init-connect='SET NAMES utf8mb4;' --innodb-flush-log-at-trx-commit=0
    environment:
      MYSQL_ALLOW_EMPTY_PASSWORD: "true"
      MYSQL_DATABASE: "rap2"
      MYSQL_ROOT_USER: "root"
      MYSQL_ROOT_PASSWORD: ""
```

## 启动

```javascript
docker-compose up -d
docker-compose exec delos node scripts/init  #第一次初始化才操作这一步.
docker exec -it rap_mysql_1 /bin/bash
mysql -u root -p
use rap2;
update Users set password='c3284d0f94606de1fd2af172aba15bf3' where fullname ='admin';
```

