# Домашнее задание к занятию 5. «Практическое применение Docker»

## Задача 1
1. Сделайте в своем github пространстве fork репозитория ```https://github.com/netology-code/shvirtd-example-python/blob/main/README.md```.   
2. Создайте файл с именем ```Dockerfile.python``` для сборки данного проекта. Используйте базовый образ ```python:3.9-slim```. Протестируйте корректность сборки. Не забудьте dockerignore.

```Dockerfile.python```
```
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt ./
RUN pip install -r requirements.txt
COPY main.py ./
CMD ["python", "main.py"]
```
![docker](https://github.com/vadimtsvetkov/-virt-04-docker-in-practice/blob/main/screenshots/Screenshot_1.png)

## Задача 3
1. Изучите файл "proxy.yaml"
2. Создайте в репозитории с проектом файл ```compose.yaml```. С помощью директивы "include" подключите к нему файл "proxy.yaml".
3. Опишите в файле ```compose.yaml``` следующие сервисы: 

- ```web```. Образ приложения должен ИЛИ собираться при запуске compose из файла ```Dockerfile.python``` ИЛИ скачиваться из yandex cloud container registry(из задание №2 со *). Контейнер должен работать в bridge-сети с названием ```backend``` и иметь фиксированный ipv4-адрес ```172.20.0.5```. Сервис должен всегда перезапускаться в случае ошибок.
Передайте необходимые ENV-переменные для подключения к Mysql базе данных по сетевому имени сервиса ```web``` 

- ```db```. image=mysql:8. Контейнер должен работать в bridge-сети с названием ```backend``` и иметь фиксированный ipv4-адрес ```172.20.0.10```. Явно перезапуск сервиса в случае ошибок. Передайте необходимые ENV-переменные для создания: пароля root пользователя, создания базы данных, пользователя и пароля для web-приложения.Обязательно используйте уже существующий .env file для назначения секретных ENV-переменных!

```compose.yaml```
```
version: '3.7'
include:
  - proxy.yaml

volumes:
  db_mysql:

services:

  db:
    image: mysql:8
    restart: on-failure
    env_file:
      - .env
    volumes:
      - /var/lib/docker/volumes/db_mysql/_data:/var/lib/mysql
    ports:
      - 3306:3306
    networks:
      backend:
        ipv4_address: 172.20.0.10

  web:
    build:
          dockerfile: Dockerfile.python
    restart: on-failure
    environment:
      - DB_HOST=db
      - DB_TABLE=requests
      - DB_USER=test_user
      - DB_NAME=test_db
      - DB_PASSWORD=test_password
    depends_on:
      - db
    networks:
      backend:
        ipv4_address: 172.20.0.5
```
2. Запустите проект локально с помощью docker compose , добейтесь его стабильной работы: команда ```curl -L http://127.0.0.1:8090``` должна возвращать в качестве ответа время и локальный IP-адрес. Если сервисы не стартуют воспользуйтесь командами: ```docker ps -a ``` и ```docker logs <container_name>```

![docker](https://github.com/vadimtsvetkov/-virt-04-docker-in-practice/blob/main/screenshots/Screenshot_2(fix).png)

5. Подключитесь к БД mysql с помощью команды ```docker exec <имя_контейнера> mysql -uroot -p<пароль root-пользователя>``` . Введите последовательно команды (не забываем в конце символ ; ): ```show databases; use <имя вашей базы данных(по-умолчанию example)>; show tables; SELECT * from requests LIMIT 10;```.

6. Остановите проект. В качестве ответа приложите скриншот sql-запроса.

![docker](https://github.com/vadimtsvetkov/-virt-04-docker-in-practice/blob/main/screenshots/Screenshot_3(fix).png)

## Задача 4
1. Запустите в Yandex Cloud ВМ (вам хватит 2 Гб Ram).
2. Подключитесь к Вм по ssh и установите docker.
3. Напишите bash-скрипт, который скачает ваш fork-репозиторий в каталог /opt и запустит проект целиком.
4. Зайдите на сайт проверки http подключений, например(или аналогичный): ```https://check-host.net/check-http``` и запустите проверку вашего сервиса ```http://<внешний_IP-адрес_вашей_ВМ>:8090```. Таким образом трафик будет направлен в ingress-proxy.
![docker](https://github.com/vadimtsvetkov/-virt-04-docker-in-practice/blob/main/screenshots/Screenshot_4.png)
6. (Необязательная часть) Дополнительно настройте remote ssh context к вашему серверу. Отобразите список контекстов и результат удаленного выполнения ```docker ps -a```
7. В качестве ответа повторите  sql-запрос и приложите скриншот с данного сервера, bash-скрипт и ссылку на fork-репозиторий.
![docker](https://github.com/vadimtsvetkov/-virt-04-docker-in-practice/blob/main/screenshots/Screenshot_5(fix).png)

```
#!/bin/bash

    sudo apt install -y docker docker-compose-v2

if [ ! -d "/opt/shvirtd-example-python" ] ; then
    sudo git clone https://github.com/vadimtsvetkov/shvirtd-example-python /opt/shvirtd-example-python
else
    cd /opt/shvirtd-example-python
    sudo git pull
fi

pwd

sudo docker compose -f compose.yaml up
```
```https://github.com/vadimtsvetkov/shvirtd-example-python```
## Задача 6
Скачайте docker образ ```hashicorp/terraform:latest``` и скопируйте бинарный файл ```/bin/terraform``` на свою локальную машину, используя dive и docker save.
Предоставьте скриншоты  действий .
![docker](https://github.com/vadimtsvetkov/-virt-04-docker-in-practice/blob/main/screenshots/Screenshot_6.png)
![docker](https://github.com/vadimtsvetkov/-virt-04-docker-in-practice/blob/main/screenshots/Screenshot_7.png)
![docker](https://github.com/vadimtsvetkov/-virt-04-docker-in-practice/blob/main/screenshots/Screenshot_8.png)
## Задача 6.1
Добейтесь аналогичного результата, используя docker cp.  
Предоставьте скриншоты  действий .
![docker](https://github.com/vadimtsvetkov/-virt-04-docker-in-practice/blob/main/screenshots/Screenshot_9.png)

