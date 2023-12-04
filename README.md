# Project_1
README for HW

## Задание: 
VM1 – установить веб-сервер Apache, интерпретатор PHP. Выбрать режим взаимодействия Apache и PHP, выбор обосновать.
VM2 – установить сервис баз данных MySQL. Настроить взаимодействие с VM1 (Настроить «двух-узловой» стек LAMP)
 Установить на полученном стенде LAMP в качестве веб-сайта CMS WordPress.
Добавить в организованный на VM1, VM2 стек LAMP второй вебсайт. Используйте CMS на ваш выбор, но не WordPress. Произвести все необходимые изменения в настройках веб- cервера. Сервис баз данных для второго вебсайта использовать также удаленный, с VM2.

В результате, оба вебсайта должны открываться с вашего компьютера в браузере по имени, но без использования внешнего DNS сервера. (Используем способ задания приоритетных DNS-записей, применяемый при локальной разработке).
Написать инструкцию по настройке в формате «Howto», как все настраивали (обоснования выбора режимов взаимодействия здесь), и добавить в отчет. «Howto» пишем в формате «инструкции будущему себе», если бы вам пришлось повторить данную процедуру через год, при этом ни разу не выполняя подобные задачи на его протяжении.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Установить 2 VM, настроить статические IP-адреса, чтобы VM работали в одной сети и проверить ping.

# Установка и настройка Apache на VM1 через apt
```
  sudo apt update
  sudo apt install apache2
```

 После завершения установки сервер Apache должен быть запущен автоматически. Проверить его статус командой
  sudo systemctl status apache2

 Установить фаервол
 
 ```
  sudo apt install ufw
```
 
 Включить фаервол UFW
 
```
  sudo ufw enable
```

 Проверить статус фаервола

```
  sudo ufw status
```

 Добавить правило, чтобы входящие запросы были разрешены извне для порта 80
 
```
  sudo ufw allow 80
```

 Проверить работу Apache
 Веб-сервер Apache должен быть доступен по адресу http://localhost в браузере. Должна отобразиться страница приветствия Apache, если все установлено и работает верно.

---------------------------------------------------------------------------------------------

# Установить интерпретатор php

```
  sudo apt update
  sudo apt install php7.4 libapache2-mod-php7.4 php-common php7.4 php7.4-cli php7.4-common php7.4-json php7.4-opcache php7.4-readline
```

Проверка модуля в Apache

```
  sudo apachectl -M                              # д.быть php7_module
  sudo a2enmod php7.4                            # добавление модуля, если его нет
  sudo a2dismod php7.4                           # disable
```

 Перезапуск Apache после внесения изменений
 ```
  sudo systemctl restart apache2
```

Проверить работу PHP
В директории /var/www/html создать файл и прописать код 

```
  sudo nano /var/www/html/info.php
  <?php phpinfo(); ?>
```

В браузере в адресной строке ввести http://localhost/info.php
В результате должны увидеть страницу с подробной информацией о конфигурации PHP.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------

# На VM2 установить MySQL/MariaDB

```
  sudo apt update
  sudo apt install mariadb-server
```

# Настройка MariaDB

```
  sudo mysql_secure_installation
```
Сделать MariaDB доступной для всех адресов

```
  sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Строку bind-address = 127.0.0.1 закомментировать, а ниже добавить

```
  bind-address = 0.0.0.0
```

Сохранить измения и перезапустить сервис

```
  sudo service mariadb restart
```

Создать 2 БД

```
  sudo mysql -u root -p
```
```
  CREATE DATABASE wordpress;
  CREATE USER 'wordpressuser'@'%' IDENTIFIED BY 'password';
  GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'%';
  FLUSH PRIVILEGES;

  CREATE DATABASE joomla;
  CREATE USER 'joomlauser'@'%' IDENTIFIED BY 'password';
  GRANT ALL PRIVILEGES ON joomla.* TO 'joomlauser'@'%';
  FLUSH PRIVILEGES;
  EXIT;
  ```
------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Установка и настройка WordPress

```
  sudo wget https://wordpress.org/latest.tar.gz
  sudo tar -zxvf latest.tar.gz
  sudo cp -R wordpress/* /var/www/html/
  sudo chown -R www-data:www-data /var/www/html/
  sudo chmod -R 755 /var/www/html/
```

Настройка виртуального хоста Apache
```
  sudo nano /etc/apache2/sites-available/wordpress.conf
```
 Вставить блок конфигурации в файл
 ```
  <VirtualHost *:80>
      ServerAdmin admin@example.com
      DocumentRoot /var/www/html/
      ServerName yourdomain.com

    <Directory /var/www/html/>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Сохранить и активировать
```
  sudo a2ensite wordpress.conf
```

Отключить стандартный виртуальный хост
```
  sudo a2dissite 000-default.conf
```

 Перезапустить Apache
 ```
  sudo service apache2 restart
```

Завершение установки через веб-интерфейс 
В браузере и перейти по адресу http://yourdomain.com (замените yourdomain.com на свое доменное имя или IP-адрес).
Следовать инструкциям на экране для завершения установки WordPress.
При настройке базы данных ввести следующие данные:
Имя базы данных: wordpress
Имя пользователя: wordpressuser
Пароль: password
Хост базы данных: localhost
Префикс таблиц: по умолчанию wp_

-----------------------------------------------------------------------------------------------------------------------------------------------------------------
# Установка и настройка CMS Joomla в качестве второго хоста

Создать файл конфигурации для второго хоста Joomla  
```
  sudo nano /etc/apache2/sites-available/joomla.conf
```

Вставить блок в файл
```
 <VirtualHost *:80>
     ServerAdmin admin@example.com
     DocumentRoot /var/www/joomla/
     ServerName joomla.example.com
     ServerAlias www.joomla.example.com

     <Directory /var/www/joomla/>
         Options FollowSymLinks
         AllowOverride All
         Require all granted
     </Directory>

     ErrorLog ${APACHE_LOG_DIR}/error.log
     CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Заменить joomla.example.com на желаемое доменное имя 
Сохранить и активировать хост
```
  sudo a2ensite joomla.conf
```

Перезапустить Apache 
```
  sudo service apache2 restart
```

Установка Joomla
```
  sudo wget https://downloads.joomla.org/ru/cms/joomla5/5-0-1/Joomla_5-0-1-Stable-Full_Package.tar.gz
  sudo tar -xf Joomla_5-0-1-Stable-Full_Package.tar.gz
  sudo cp -R joomla/* /var/www/html/
  sudo chown -R www-data:www-data /var/www/joomla/
  sudo chmod -R 755 /var/www/joomla/
```

Завершение установки через веб-интерфейс 
В браузере и перейти по адресу http://joomla.example.com (заменить joomla.example.com на свое доменное имя для Joomla).
Следовать инструкциям на экране для завершения установки Joomla.
При настройке базы данных ввести следующие данные:
Тип базы данных: MySQLi
Имя сервера базы данных: localhost
Имя пользователя базы данных: joomlauser
Пароль базы данных: password
Префикс таблиц: по умолчанию jos_ (или выберать другой префикс по своему усмотрению)

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

Инструкция преподавателя:

README 1

 Установим веб-сервер Apache используя возможности стандартного пакетного менеджера, при необходимости решим конфликты запуска
```
sudo apt update
sudo apt install apache2
```

 После окончания установки откроем стандартную приветственную страницу веб-сервера в браузере
http://192.168.1.32/

 Проверим конфигурацию веб-сервера, выясним, где расположен документ, соответствующий приветственной странице веб-сервера
 editing its Virtual Host file found in 
/etc/apache2/sites-enabled/000-default.conf
 move to
vim /etc/apache2/sites-enabled/000-default.conf
 find VHost
 We can modify its content in 
/var/www/html

 Добавим/создадим на сервере простой документ формата HTML (образец документа и формата можно найти в Интернет https://www.w3schools.com/html/), содержащий некоторый текст, например "Hello from HTTP Server", запомним путь (URL), по которому мы можем обращаться к этому документу на сервере
```
vim test.html
<!DOCTYPE html>
<html>
<head>
<title>Page Title</title>
</head>
<body>

<h1>This is a Heading</h1>
<p>Hello from HTTP Server</p>

</body>
</html> 
```
```
vim index.html
 <a href="test.html" target="_blank">Opening test.html</a> 
```
 # ----------------------------------------
  Установим сервер баз данных MySQL/Mariadb, а так же cli-инструмент работы с базой данных mysql используя возможности стандартного пакетного менеджера.
```
sudo apt install wget
wget https://dev.mysql.com/get/mysql-apt-config_0.8.22-1_all.deb
sudo apt install ./mysql-apt-config_0.8.22-1_all.deb
sudo apt update
```
```
sudo apt install mysql-server
sudo service mysql status
sudo apt install mysql-client
```
 Подключимся к нашей базе данных с помощью cli-инструмента, выведем список существующих баз данных.
```
mysql -u root -p   (qwerty123)
show databases;
or
mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'qwerty';
mysql -u root -pqwerty
```

Создадим новую базу данных.
```
CREATE DATABASE myfirstdb;
USE myfirstdb;
```
 Создадим новую таблицу со структурой по выбору (3-5 полей).
```
SHOW TABLES;
CREATE TABLE towns (name VARCHAR(20), district VARCHAR(20), owner_fio VARCHAR(20), start DATE);
DESCRIBE 
```
 Добавим две записи  в новую таблицу.
```
LOAD DATA LOCAL INFILE '/path/towns.txt' INTO TABLE towns;
INSERT INTO towns VALUES ('Kobrin','Brest','Sidorov D.D.','1356-01-01');
```
 Сделаем выборку данных по всей таблице
```
SELECT * FROM towns;
```
 Сделаем резервную копию данных в базе в файл на файловой системе сервера (mysqldump)
```
mysqldump -uroot -p --all-databases > dump.sql
```
 Удалим первую запись
```
DELETE FROM towns as t WHERE t.name='Kobrin';
```
 Удалим созданную таблицу.
```
DROP TABLE towns;
```
 Удалим созданную базу данных
```
DROP DATABASE myfirstdb;
```
 Удалим сервер баз данных MySQL/Mariadb c данной машины.
```
sudo apt purge mysql-server
```

README 2

  Установим интерпретатор PHP, используя возможности стандартного пакетного менеджера.
```
sudo apt update
sudo apt install php7.4 libapache2-mod-php7.4 php-common php7.4 php7.4-cli php7.4-common php7.4-json php7.4-opcache php7.4-readline
​
```

Запустим интерпретатор.
Проверка модуля в Apache
```
  sudo apachectl -M # д.быть php7_module
  sudo a2enmod php7.4 #add module
  sudo a2dismod php7.4 #disable
```
 После изменений - рестарт apache
  ```
  sudo systemctl restart apache2
  sudo vim /var/www/html/info.php
    <?php phpinfo();?>
​  
```

http://192.168.1.32/info.php
Создадим на сервере простейший PHP-файл (пример https://www.php.net/manual/en/tutorial.firstpage.php)
Изменим его содержимое на "Hello IT-Academy"
Передадим интерпретатору PHP параметром данный файл, изучим вывод.
php first.php
​
Создадим для нашего веб-сервера второй виртуальный хост.
```
cd  /etc/apache2/sites-available/
sudo cp 000-default.conf second.conf
```
Назначим нашим виртуальным хостам различные имена.
```
sudo vim second.conf
    ServerAdmin vpeshkur@debian.by
	DocumentRoot /var/www/second
sudo vim 000-default.conf
```
Внесем имена в локальный файл конфигурации ДНС hosts на вашем рабочем компьютере.
```
sudo vim /etc/hosts
```
​
Разместим в новом виртуальном хосте нашу дополнительную страницу из первого шага практики.
```
sudo mkdir /var/www/second
cp /var/www/html/test.html /var/www/second/index.html
vim /var/www/second/index.html
​
```

```
sudo a2ensite second.conf
sudo systemctl reload apache2
```
	
Убедимся, что приветственная страница веб-сервера и дополнительная страница открываются по разным именам веб-сервера
www.second.by
first.by
​
Переведем второй виртуальный хост в режим работы как по HTTP так и по HTTPS
Сгенерируем и подключим в настройку веб-сервера само-подписанный SSL-сертификат для второго виртуального хоста
```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/Second-apache-selfsigned.key -out /etc/ssl/certs/Second-apache-selfsigned.crt
sudo openssl x509 -text -noout -in /etc/ssl/certs/apache-selfsigned.crt #Просмотр серта
```

```
sudo vim /etc/apache2/sites-available/second.conf
<VirtualHost *:443>
   ServerName www.second.by
   DocumentRoot /var/www/second
​
   SSLEngine on
   SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
   SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
</VirtualHost>
```
активировать файл конфигурации
```
sudo a2ensite second.conf
```
check errors
```
sudo apache2ctl configtest
Invalid command 'SSLEngine', perhaps misspelled or defined by a module not included in the server configuration
sudo a2enmod ssl
sudo systemctl reload apache2
sudo apache2ctl configtest
```

Убедимся, что веб-сервер для второго виртуального хоста по протоколу HTTPS отдает наш сертификат
https://second.by/
​
Настроим корректное перенаправление с HTTP на HTTPS для второго виртуального хоста
​
```
<VirtualHost *:80>
	ServerName www.second.by
	Redirect / https://second.by/
</VirtualHost>
```
```
sudo apache2ctl configtest
sudo systemctl reload apache2
```


Проверим конфигурацию веб-сервера, выясним, где расположен документ, соответствующий приветственной странице веб-сервера
editing its Virtual Host file found in 
```
/etc/apache2/sites-enabled/000-default.conf
```
move to
```
vim /etc/apache2/sites-enabled/000-default.conf
```

find VHost
We can modify its content in 
/var/www/html

Добавим/создадим на сервере простой документ формата HTML (образец документа и формата можно найти в Интернет https://www.w3schools.com/html/), содержащий некоторый текст, например "Hello from HTTP Server", запомним путь (URL), по которому мы можем обращаться к этому документу на сервере
```
vim test.html
<!DOCTYPE html>
<html>
<head>
<title>Page Title</title>
</head>
<body>

<h1>This is a Heading</h1>
<p>Hello from HTTP Server</p>

</body>
</html> 

vim index.html
 <a href="test.html" target="_blank">Opening test.html</a> 
```


Создадим для нашего веб-сервера второй виртуальный хост.
```
cd  /etc/apache2/sites-available/
sudo cp 000-default.conf second.conf
```

Назначим нашим виртуальным хостам различные имена.
```
sudo vim second.conf
    ServerAdmin vpeshkur@debian.by
	DocumentRoot /var/www/second
sudo vim 000-default.conf
```

Внесем имена в локальный файл конфигурации ДНС hosts на вашем рабочем компьютере.
```
sudo vim /etc/hosts
​```

Разместим в новом виртуальном хосте нашу дополнительную страницу из первого шага практики.

```
sudo mkdir /var/www/second
cp /var/www/html/test.html /var/www/second/index.html
vim /var/www/second/index.html
​```
```
sudo a2ensite second.conf
sudo systemctl reload apache2
```
	
Убедимся, что приветственная страница веб-сервера и дополнительная страница открываются по разным именам веб-сервера
www.second.by
first.by







  
