# Домашнее задание к занятию «Репликация и масштабирование. Часть 1» - Одинцов Игорь

### Задание 1
На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.
*Ответить в свободной форме.*

### Ответ
При режиме master-slave все манипуляции с данными производятся на "мастере", а "слэйв" реплицирует внесённые изменения с "мастера".

Режим master-master по сути тот же master-slave, где "мастеру" добавляется роль "слэйва", и он будет реплицировать данные с того сервера, что был "слэйвом". Работа в данном режиме - нежелательна, т.к. может привести к потерям данных.

---

### Задание 2
Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.
*Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.*

### Ответ

* По порядку нв Мастере и Слейве:
```ruby
wget https://dev.mysql.com/get/mysql80-community-release-el7-6.noarch.rpm
```
```ruby
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
```
```ruby
rpm -Uvh mysql80-community-release-el7-6.noarch.rpm
```
```ruby
yum -y install mysql-server mysql-client
```
```ruby
mysqld --initialize
```
```ruby
chown -R mysql: /var/lib/mysql
```
```ruby
mkdir -p /var/log/mysql
```
```ruby
chown -R mysql: /var/log/mysql
```

* Редактируем /etc/my.cnf:
```ruby
bind-address=0.0.0.0
server-id=(1..n)
log_bin=/var/log/mysql/mybin.log
```

* Проверяем пароль присвоенный при инициализации BD:
```ruby
cat /var/log/mysqld.log
```

* Сменить пароль:
```ruby
mysql -p
```
```ruby
ALTER USER 'root'@'localhost' IDENTIFIED BY '12345';
```
```ruby
FLUSH PRIVILEGES;
```

* На мастере:
```ruby
CREATE USER 'replication'@'%' IDENTIFIED WITH mysql_native_password BY 'Repl11Pass!';
GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';
```
```ruby
SHOW MASTER STATUS;
```
![Z2 1](https://github.com/Bestenar/12.6-Replik_Mashtab-1-hw/assets/111271419/471d514a-53d0-4ff7-b36a-0206ac416aad)

* На слэйве:
```ruby
CHANGE MASTER TO MASTER_HOST='192.168.*.*', MASTER_USER='replication', MASTER_PASSWORD='Repl11Pass!', MASTER_LOG_FILE = 'mybin.000001', MASTER_LOG_POS = (число из колонки position из статуса мастера);
```
```ruby
START SLAVE;
```
```ruby
SHOW SLAVE STATUS\G;
```
![Z2 2](https://github.com/Bestenar/12.6-Replik_Mashtab-1-hw/assets/111271419/300bd3c2-4e9a-4d6e-ad3d-ef61d59d53af)

* На мастере:
```ruby
create database hw_12_6_test;
```
* На слэйве:
```ruby
show databases;
```
![Z2 3](https://github.com/Bestenar/12.6-Replik_Mashtab-1-hw/assets/111271419/e01e0396-bad0-4849-9e6f-19a59fc47b22)
