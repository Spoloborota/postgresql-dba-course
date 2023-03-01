# Установка и настройка PostgteSQL в контейнере Docker

Использовал уже существующий у меня Docker Engine в MacOS, и уже крутящийся контейнер с докером, который я использую для локальной работы:
> PostgreSQL 14.5 (Debian 14.5-1.pgdg110+1) on aarch64-unknown-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit

Использованный образ из докер-композа:
```
postgres:
    image: postgres:14
    volumes:
        - ./_data/postgres/var/lib/postgresql/data:/var/lib/postgresql/data
    environment:
        POSTGRES_PASSWORD: "secret"
    ports:
        - "25432:5432"
    healthcheck:
        test: ["CMD-SHELL", "pg_isready -U postgres"]
        interval: 10s
        timeout: 1s
        retries: 3
        start_period: 10s
```

набросал образ для клиента постгреса:
```
FROM alpine:3.15
RUN apk --no-cache add postgresql14-client
ENTRYPOINT [ "/bin/sh" ]
```

добавил его в докер-компот:
```
pg_client:
    build:
    context: .
    dockerfile: PgClientDockerfile
```

сбилдил образ:
```
docker-compose build pg_client
```

запустил контейнер, зашел в шелл внутри контейнера:
```
docker-compose run --rm pg_client -i
```

подключился к постгресу:
```
/ # psql -h postgres -U postgres
Password for user postgres:
psql (14.7, server 14.5 (Debian 14.5-1.pgdg110+1))
Type "help" for help.
```

глянул какие базки уже есть:
```
postgres=# \l
List of databases
Name       |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-----------------+----------+----------+------------+------------+-----------------------
faceeee         | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
postgres        | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
styles          | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
styles_dump     | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
subs_service    | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
template0       | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
|          |          |            |            | postgres=CTc/postgres
template1       | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
|          |          |            |            | postgres=CTc/postgres
user_service    | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
webhook_service | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
(9 rows)
```

создал там таблицу **persons** из прошлого урока и добавил 2 записи.

подключился DBeaver'ом к постгресу по порту 25432 с ноута, где и крутится docker engine, проверил, что новая таблица и записи в ней на месте.

удалил контейнер с постгресом:
```
➜  docker docker ps
CONTAINER ID   IMAGE                   COMMAND                  CREATED        STATUS                 PORTS                                               NAMES
9218c20038c1   redis:6.0               "docker-entrypoint.s…"   2 months ago   Up 3 weeks (healthy)   0.0.0.0:26379->6379/tcp                             docker_redis_1
8e00b2150137   localstack/localstack   "docker-entrypoint.sh"   4 months ago   Up 3 weeks (healthy)   4510-4559/tcp, 5678/tcp, 127.0.0.1:4566->4566/tcp   localstack
0f7d4b3e1600   postgres:14             "docker-entrypoint.s…"   4 months ago   Up 4 weeks (healthy)   0.0.0.0:25432->5432/tcp                             docker_postgres_1
➜  docker docker stop 0f7d4b3e1600 && docker rm 0f7d4b3e1600
0f7d4b3e1600
0f7d4b3e1600
➜  docker docker ps                                         
CONTAINER ID   IMAGE                   COMMAND                  CREATED        STATUS                 PORTS                                               NAMES
9218c20038c1   redis:6.0               "docker-entrypoint.s…"   2 months ago   Up 3 weeks (healthy)   0.0.0.0:26379->6379/tcp                             docker_redis_1
8e00b2150137   localstack/localstack   "docker-entrypoint.sh"   4 months ago   Up 3 weeks (healthy)   4510-4559/tcp, 5678/tcp, 127.0.0.1:4566->4566/tcp   localstack
```

запускал некорректно контейнер докер-композа через:
```
docker-compose run -d postgres
```

запустил корректно контейнер:
```
docker-compose up -d postgres
```

подконнектился DBeaver'ом снова и увидел все свои данные на прежнем месте.