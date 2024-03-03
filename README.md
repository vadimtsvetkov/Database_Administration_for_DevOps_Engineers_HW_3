![Kali](https://img.shields.io/badge/Kali-268BEE?style=for-the-badge&logo=kalilinux&logoColor=white) ![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white) ![MySQL](https://img.shields.io/badge/mysql-%2300f.svg?style=for-the-badge&logo=mysql&logoColor=white)
# Домашнее задание к занятию 3. «MySQL»

## Задача 1

#### Используя Docker, поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

- Создаем файл docker-compose.yml
```sh
docker-compose.yml

services:
  db:
    image: mysql:8
    container_name: db
    environment:
      MYSQL_DATABASE: "test_db"
      MYSQL_USER: "test_admin_user"
      MYSQL_PASSWORD: "admin"
      MYSQL_ROOT_PASSWORD: "root"
    volumes:
      - ./mysql:/var/lib/mysql
      # Сразу монтируем дамп БД
      - /home/szangiev/hw_db:/dump
    ports:
      - "3306:3306"
```
- Поднимаем контейнер с помощью команды
```sh
docker compose -f docker-compose.yml up
```
![img](https://github.com/vadimtsvetkov/Database_Administration_for_DevOps_Engineers_HW_3/blob/main/Screenshots/docker-compose%20up.png)

#### Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-03-mysql/test_data) и восстановитесь из него. Перейдите в управляющую консоль `mysql` внутри контейнера.
- Проверяем, что контейнер запущен и посмотрим его id
```sh
sudo docker ps
```
![img](https://github.com/vadimtsvetkov/Database_Administration_for_DevOps_Engineers_HW_3/blob/main/Screenshots/docker%20ps.jpg)

- Подключаемся к терминалу контейнера по его id
```sh
docker exec -it dc3e783eac5d bash
```
- Импортируем дамп в БД
- Подключаемся к БД
```sh
mysql -u test_admin_user -p test_db < dump/test_dump.sql
mysql -u test_admin_user -p
```
![img](https://github.com/vadimtsvetkov/Database_Administration_for_DevOps_Engineers_HW_3/blob/main/Screenshots/mysql.png)

#### Найдите команду для выдачи статуса БД и **приведите в ответе** из её вывода версию сервера БД.

![img](https://github.com/vadimtsvetkov/Database_Administration_for_DevOps_Engineers_HW_3/blob/main/Screenshots/mysql%20's'.png)

#### **Приведите в ответе** количество записей с `price` > 300.
```sh
SELECT COUNT(*) FROM orders WHERE price > 300;
```
![img](https://github.com/vadimtsvetkov/Database_Administration_for_DevOps_Engineers_HW_3/blob/main/Screenshots/select.png)

В следующих заданиях мы будем продолжать работу с этим контейнером.

## Задача 2

#### Создайте пользователя test в БД c паролем test-pass, используя:

- плагин авторизации mysql_native_password
- срок истечения пароля — 180 дней 
- количество попыток авторизации — 3 
- максимальное количество запросов в час — 100
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James".

```sh
CREATE USER 'test'@'%' IDENTIFIED WITH mysql_native_password BY 'test-pass' 
PASSWORD EXPIRE INTERVAL 180 DAY 
FAILED_LOGIN_ATTEMPTS 3 
ATTRIBUTE '{"name": "James", "last_name": "Pretty"}';
```

#### Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.

```sh
GRANT SELECT ON test_db.* TO 'test'@'%';
```

#### Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES, получите данные по пользователю `test`.

```sh
SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES WHERE USER='test';
```
#### Приведите в ответе к задаче.

![img](https://github.com/vadimtsvetkov/Database_Administration_for_DevOps_Engineers_HW_3/blob/main/Screenshots/slqN2.png)
## Задача 3

#### Установите профилирование `SET profiling = 1`.

```sh
SET profiling = 1;
```

#### Изучите вывод профилирования команд `SHOW PROFILES;`.

```sh
SHOW PROFILES;
```

![img](https://github.com/vadimtsvetkov/Database_Administration_for_DevOps_Engineers_HW_3/blob/main/Screenshots/sql3.1.png)

#### Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

```sh
SHOW TABLE STATUS LIKE 'orders';
```

![img](https://github.com/vadimtsvetkov/Database_Administration_for_DevOps_Engineers_HW_3/blob/main/Screenshots/sql3.2.png)

#### Измените `engine`.
- на `MyISAM`,
- на `InnoDB`.

```sh
ALTER TABLE orders ENGINE = InnoDB;
SET profiling = 1;
SHOW PROFILES;
ALTER TABLE orders ENGINE = MyISAM;
SET profiling = 1;
SHOW PROFILES;
```

#### Приведите время выполнения и запрос на изменения из профайлера в ответе

![img](https://github.com/vadimtsvetkov/Database_Administration_for_DevOps_Engineers_HW_3/blob/main/Screenshots/sql3.3.png)

## Задача 4 

#### Изучите файл `my.cnf` в директории /etc/mysql. Измените его согласно ТЗ (движок InnoDB):

- скорость IO важнее сохранности данных;
- нужна компрессия таблиц для экономии места на диске;
- размер буффера с незакомиченными транзакциями 1 Мб;
- буффер кеширования 30% от ОЗУ;
- размер файла логов операций 100 Мб.

#### Приведите в ответе изменённый файл `my.cnf`.

```sh
[mysqld]
innodb_flush_log_at_trx_commit = 2
innodb_file_per_table = 1
innodb_buffer_pool_size = 30% of available memory
innodb_log_file_size = 100M

sync_binlog = 0
innodb_flush_method = O_DIRECT

innodb_log_buffer_size = 1M
innodb_compression_algorithm = lz4
innodb_compression_level = 6
```
