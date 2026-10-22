# Отчёт по домашнему заданию: MySQL 8.0 в Docker, пользователи, восстановление дампа Sakila

## Задание 1

### 1.1. Поднятие чистого инстанса MySQL 8.0+ в Docker

Контейнер запущен командой:

```bash
docker run --name mysql-hw -e MYSQL_ROOT_PASSWORD=rootpassword -p 3306:3306 -d mysql:8.0
Версия сервера, подтверждённая при подключении: Server version: 8.0.46 MySQL Community Server - GPL.

1.2. Создание учётной записи sys_temp
sql
CREATE USER 'sys_temp'@'localhost' IDENTIFIED BY 'password';
1.3. Список пользователей в базе данных
sql
SELECT User, Host FROM mysql.user;
https://screenshot-1.png/

1.4. Выдача всех прав пользователю sys_temp
sql
GRANT ALL PRIVILEGES ON *.* TO 'sys_temp'@'localhost';
FLUSH PRIVILEGES;
1.5. Список прав пользователя sys_temp
sql
SHOW GRANTS FOR 'sys_temp'@'localhost';
https://screenshot-2.png/

1.6. Переподключение от имени sys_temp
Выполнен выход из сеанса root и вход под новой учётной записью:

bash
docker exec -it mysql-hw mysql -u sys_temp -p
# Enter password: password
Для совместимости тип аутентификации переключён на mysql_native_password:

sql
ALTER USER 'sys_temp'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
Подключение успешно: Your MySQL connection id is 11, Server version: 8.0.46.

1.7. Восстановление дампа базы данных
Дамп Sakila скачан и распакован на хосте, файлы скопированы в контейнер и загружены через SOURCE:

bash
wget https://downloads.mysql.com/docs/sakila-db.zip
unzip sakila-db.zip
docker cp sakila-db/sakila-schema.sql mysql-hw:/tmp/
docker cp sakila-db/sakila-data.sql mysql-hw:/tmp/
sql
SOURCE /tmp/sakila-schema.sql;
SOURCE /tmp/sakila-data.sql;
1.8. Список таблиц восстановленной базы
sql
USE sakila;
SHOW TABLES;
Получено 23 строки (таблицы и представления Sakila).

https://screenshot-3.png/

Задание 2. Таблица «таблица — первичный ключ»
Реальные таблицы (BASE TABLE)
Таблица	Первичный ключ
actor	actor_id
address	address_id
category	category_id
city	city_id
country	country_id
customer	customer_id
film	film_id
film_actor	actor_id, film_id (составной)
film_category	film_id, category_id (составной)
film_text	film_id
inventory	inventory_id
language	language_id
payment	payment_id
rental	rental_id
staff	staff_id
store	store_id
Представления (VIEW) — первичного ключа нет
Таблица	Первичный ключ
actor_info	— (VIEW)
customer_list	— (VIEW)
film_list	— (VIEW)
nicer_but_slower_film_list	— (VIEW)
sales_by_film_category	— (VIEW)
sales_by_store	— (VIEW)
staff_list	— (VIEW)
Проверка типов объектов и первичных ключей выполнена через information_schema:

sql
SELECT TABLE_NAME, TABLE_TYPE
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'sakila'
ORDER BY TABLE_TYPE, TABLE_NAME;

SELECT TABLE_NAME, COLUMN_NAME, ORDINAL_POSITION
FROM information_schema.KEY_COLUMN_USAGE
WHERE TABLE_SCHEMA = 'sakila'
  AND CONSTRAINT_NAME = 'PRIMARY'
ORDER BY TABLE_NAME, ORDINAL_POSITION;
https://screenshot-4.png/

Выводы
В ходе работы освоены: запуск MySQL 8.0 в Docker, создание пользователя и выдача прав, переключение типа аутентификации, восстановление дампа через SOURCE внутри контейнера, анализ структуры базы через information_schema и разделение таблиц и представлений при определении первичных ключей.