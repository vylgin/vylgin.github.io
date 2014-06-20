---
layout: post
title: "Как я импортировал CSV в MongoDB"
date: 2014-06-17 04:03:03 -0400
comments: true
categories: [mongodb]
---

В глаза попался дамп базы MySQL с структурированными данными. "Это хорошая возможность познакомиться с MongoDB" - подумал я. 

Ниже я расскажу о том, как экспортировал данные из дампа MySQL в MongoDB через промежуточный CSV файл.

<!-- more -->

Я развернул mysql-сервер, залил в него дамп и сделал экспорт данных в CSV файл. О том как это сделать полно информации, так что подробно описывать не буду, в силу зрелости MySQL.

#### 1. Установка MongoDB на ubuntu 14.04

Процесс установки MongoDB - это выполнение четырех команд из [официального туториала](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/):


1. Импортирование публичного ключа

        sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
        
2. Создание файла mongodb.list

        echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | sudo tee /etc/apt/sources.list.d/mongodb.list

3. Обновление локальных пакетов

        sudo apt-get update
        
4. Установка последней стабильной версии MongoDB

        sudo apt-get install mongodb-org
        
Теперь мы можем запускать, останавливать и перезапускать нашу базу командами:

    sudo service mongod start
    sudo service mongod stop
    sudo service mongod restart
    
Запустить консольный клиент к базе можно командой `mongo`.
    
#### 2. Добавление админа БД

##### Добавление админа для всей БД

Для добавления учетных записей используется команда `db.createUser()`, чтобы создать учетную запись администратора для всех баз нужно выполнить:

    use admin
    db.createUser(
      {
        user: "siteUserAdmin",
        pwd: "password",
        roles:
        [
          {
            role: "userAdminAnyDatabase",
            db: "admin"
          }
        ]
      }
    )
    
Чтобы включить вход по учетным записям, в конфигурационном файле `/etc/mongod.conf ` нужно раскомментировать строку:

    auth = true
    
Для коннекта к базе нужно выполнить команду:

    mongo -u siteUserAdmin -p

##### Добавление админа для локальной базы

Чтобы создать учетную запись администратора для конкретной базы, нужно выполнить команду `db.createUser()`:

    db.createUser(
      {
        user: "db1_admin",
        pwd: "password",
        roles:
        [
          {
            role: "userAdmin",
            db: "db1"
          }
        ]
      }
    )
    
Чтобы админ мог читать и писать из базы, добавим ему соответствующую роль:
    
    db.grantRolesToUser(
      "db1_admin",
      [
        {
          role: "readWrite", db: "db1"
        }
      ]
    )

Войти в базу можно командой:
    
    mongo -u db1_admin -p password db1
    
##### Импорт CSV файла

Импортирование выполняется командой `mongoimport`. Пусть у нас есть файл `file.csv`, тогда импортировать его в MongoDB можно так:

    mongoimport --db db1 --type csv --headerline --file file.csv 

Или так, если уже создана учетная запись для базы:

    mongoimport --db db1 --type csv --headerline --file file.csv -u db1_admin -p
