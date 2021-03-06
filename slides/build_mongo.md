# Mongodb

MongoDB - это нереляционное хранилище данных, schema-less.

Мы установим Mщтпщ внутрь контейнера на ALpine Linux

## Шаг 1 - сборка образа.

Нам нужен образ с дистрибутивом Alpine Linux и Mongo. Создадим соотвествующий docker-file:

<pre>
FROM alpine:edge

#MONGO
ENV APP_MONGO_HOST mongodb
ENV APP_MONGO_PORT 27017

EXPOSE $BIND_PORT


RUN echo "http://dl-8.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories

RUN apk update && apk upgrade && apk --no-cache --update-cache add gcc gfortran python python-dev py-pip build-base wget\
    && apk add mongodb su-exec
# mongodb@edge=3.
</pre>

Соберём образ с этим докер-файлом и запусти в фоновом режиме

<pre>
    sudo docker-compose --project-name mongo-client -f docker-compose.yml up --build -d
</pre>

Если проблемы с сетью 

Может возникнуть непонятная сетевая ошибка
<pre>
Creating network "postgresclient_default" with the default driver
ERROR: could not find an available, non-overlapping IPv4 address pool among the defaults to assign to the network
</pre>

попробуйте
<pre>
ifconfig

выбрать подозриттельный сетевой интерфейс tun0 и удалить его

sudo ip link delete tun0
</pre>

Если не поднимается - нужно проверить, возможно остались следы от старых образов:

<pre>
$ sudo docker-compose --project-name mongo-client -f docker-compose.yml ps
                                  Name                                               Command              State       Ports
-----------------------------------------------------------------------------------------------------------------------------
a93c0a29e67e_a93c0a29e67e_a93c0a29e67e_mongoclient_alpine-mongo-client_1   /bin/sh                       Exit 137
mongoclient_mongodb_1                                                      docker-entrypoint.sh mongod   Up         27017/tcp

$  sudo docker-compose --project-name mongo-client -f docker-compose.yml rm --all;

</pre>

Или перезапустить compose
<pre>
    docker-compose -f docker-compose.yml down

    docker-compose -f docker-composeyml up
</pre>

Если build выполнился - смотрим, что всё ок

<pre>
$ sudo docker-compose --project-name mongo-client -f docker-compose.yml up -d
Starting mongoclient_mongodb_1 ...
Starting mongoclient_mongodb_1 ... done
Starting mongoclient_alpine-mongo-client_1 ...
Starting mongoclient_alpine-mongo-client_1 ... done
adzhumurat@adzhumurat-HP-G5:~/PycharmProjects/flask_docker_app/mongo_interactions$ sudo docker-compose --project-name mongo-client -f docker-compose.yml ps
              Name                            Command             State     Ports
-----------------------------------------------------------------------------------
mongoclient_alpine-mongo-client_1   /bin/sh                       Up
mongoclient_mongodb_1               docker-entrypoint.sh mongod   Up      27017/tcp
</pre>

В консоли побежит информация о сборке, после сообщения об удачном завершении можно проверить, что все сервисы поднялись успешно

<pre>
adzhumurat@adzhumurat-HP-G5:~$ sudo docker ps
CONTAINER ID        IMAGE                             COMMAND                  CREATED             STATUS              PORTS                      NAMES
28db6d9c9b72        mongoclient_alpine-mongo-client   "/bin/sh"                11 minutes ago      Up 11 minutes                                  mongoclient_alpine-mongo-client_1
41bdd3a841ef        mongo:latest                      "docker-entrypoint.s…"   11 minutes ago      Up 11 minutes       0.0.0.0:27017->27017/tcp   mongoclient_mongodb_1
</pre>

Образ собрался, можем подключаться
<pre>
sudo docker-compose --project-name mongo-client -f docker-compose.yml run --rm alpine-client
</pre>

Попробуем подключиться к Mongo
<pre>
    # /usr/bin/mongo -host $APP_MONGO_HOST -port $APP_MONGO_PORT
</pre>

ЕСли что-то пошло не так, можно посмотреть в логи
<pre>
adzhumurat@adzhumurat-HP-G5:~$ sudo docker ps
CONTAINER ID        IMAGE                             COMMAND                  CREATED             STATUS              PORTS               NAMES
4fb59e3c1685        mongoclient_alpine-mongo-client   "/bin/sh"                4 minutes ago       Up 4 minutes                            sleepy_heyrovsky
699ad0a36bfc        mongoclient_alpine-mongo-client   "/bin/sh"                8 minutes ago       Up 4 minutes                            mongoclient_alpine-mongo-client_1
fb743b409812        mongo:latest                      "docker-entrypoint.s…"   20 minutes ago      Up 4 minutes        27017/tcp           mongoclient_mongodb_1
1cb26b58e8c8        postgres                          "docker-entrypoint.s…"   3 days ago          Up 3 days           5432/tcp            netology-postgres
adzhumurat@adzhumurat-HP-G5:~$ sudo docker logs fb743b409812 | tail
2018-07-23T08:30:09.608+0000 I FTDC     [initandlisten] Initializing full-time diagnostic data capture with directory '/data/db/diagnostic.data'
2018-07-23T08:30:09.609+0000 I NETWORK  [initandlisten] waiting for connections on port 27017
adzhumurat@adzhumurat-HP-G5:~$
</pre>

Рекомендую так же поудалять все контейнеры, которые завершились

<pre>
    sudo docker ps -a | grep Exit | cut -d ' ' -f 1 | xargs sudo docker rm
</pre>
