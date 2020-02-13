# Инструкция по использованию консольных клиентов для СУБД MySQL и PostgreSQL

## Docker Compose

В этом примере рассматривается следующая конфигурация `docker-compose.yml`:

```
version: '3'
services:
  mysqldb:
    image: mysql:latest
    ports:
      - '3306:3306'
    environment:
      - MYSQL_RANDOM_ROOT_PASSWORD=yes
      - MYSQL_DATABASE=app
      - MYSQL_USER=app
      - MYSQL_PASSWORD=pass
  postgresdb:
    image: postgres:latest
    ports:
      - '5432:5432'
    environment:
      - POSTGRES_DB=app
      - POSTGRES_USER=app
      - POSTGRES_PASSWORD=pass
```

Необходимо запустить оба сервиса через `docker-compose up`.

Удостоверяемся, что оба сервиса запущены:
```
docker-compose ps

      Name                    Command              State                 Ports
--------------------------------------------------------------------------------------------
sql_mysqldb_1      docker-entrypoint.sh mysqld     Up      0.0.0.0:3306->3306/tcp, 33060/tcp
sql_postgresdb_1   docker-entrypoint.sh postgres   Up      0.0.0.0:5432->5432/tcp

```

Docker Compose позволяет подключаться к каждому сервису и выполнять команды внутри него.

В образах MySQL и Postgres идут в комплекте CLI клиенты для СУБД (т.е.) возможность из командной строки выполнять запросы в СУБД.

В MySQL он называется `mysql`, в Postgres - `psql`.

Для подключения используем [`exec`](https://docs.docker.com/compose/reference/exec/)

### MySQL

```
docker-compose exec mysqldb mysql -u app app -p 
```

где `-u app` имя пользователя, `app` база данных, `-p` указание на подключение с паролем.

Важно: пароль при наборе отображаться не будет, поэтому вводите "вслепую";

После чего вы увидите следующее:

```
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.19 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

Полное описание клиента приведено [на странице документации](https://dev.mysql.com/doc/refman/8.0/en/mysql.html).

Мы приведём лишь ключевые команды:
* `SHOW TABLES;` - показывает существующие в базе таблицы
* `DESCRIBE accounts;` - показывает информацию о колонках таблицы
* `help` - справка
* `exit` - выход из клиента
* здесь же вы можете выполнять все команды SQL (вроде `CREATE`, `DROP`, `SELECT`, `INSERT`, `DELETE` и т.д.).

### Postgres

```
docker-compose exec postgresdb psql -U app -d app -W 
```

где `-u app` имя пользователя, `-d app` база данных, `-W` указание на подключение с паролем.

Важно: пароль при наборе отображаться не будет, поэтому вводите "вслепую";

После чего вы увидите следующее:

```
psql (12.1 (Debian 12.1-1.pgdg100+1))
Type "help" for help.

app=#
```

Полное описание клиента приведено [на странице документации](https://www.postgresql.org/docs/current/app-psql.html).

Мы приведём лишь ключевые команды:
* `\d` - показывает существующие в базе таблицы
* `\d+ accounts;` - показывает информацию о колонках таблицы
* `help` - справка
* `exit` - выход из клиента
* здесь же вы можете выполнять все команды SQL (вроде `CREATE`, `DROP`, `SELECT`, `INSERT`, `DELETE` и т.д.).
