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
                                            
services:                                   
                                            
  db:                                       
    image: mysql:8                          
    command: --default-authentication-plugin
    restart: on-failure                          
    environment:                            
      MYSQL_ROOT_PASSWORD: 12345            
      MYSQL_DATABASE: db1                   
      MYSQL_USER: user                      
      MYSQL_PASSWORD: 12345         
    volumes:                                
      - ./docker_volumes/mysql:/var/lib/mysq
    networks:                               
      backend:                              
        ipv4_address: 172.20.0.10           
                                            
  web:                                      
    build:                                  
          dockerfile: Dockerfile.python     
    restart: on-failure                     
    environment:                            
      - DB_HOST=172.20.0.10                 
      - DB_TABLE=requests                   
      - DB_USER=root                        
      - DB_NAME=db1                         
      - DB_PASSWORD=12345                   
    depends_on:                             
      - db                                  
    ports:                                  
      - 5000:5000                           
    networks:                               
      backend:                              
        ipv4_address: 172.20.0.5            
                                            
networks:                                   
  backend:                                  
    driver: bridge                          
    ipam:                                   
      config:                               
      - subnet: 172.20.0.0/24
```
2. Запустите проект локально с помощью docker compose , добейтесь его стабильной работы: команда ```curl -L http://127.0.0.1:8090``` должна возвращать в качестве ответа время и локальный IP-адрес. Если сервисы не стартуют воспользуйтесь командами: ```docker ps -a ``` и ```docker logs <container_name>```

![docker](https://github.com/vadimtsvetkov/-virt-04-docker-in-practice/blob/main/screenshots/Screenshot_2.png)

5. Подключитесь к БД mysql с помощью команды ```docker exec <имя_контейнера> mysql -uroot -p<пароль root-пользователя>``` . Введите последовательно команды (не забываем в конце символ ; ): ```show databases; use <имя вашей базы данных(по-умолчанию example)>; show tables; SELECT * from requests LIMIT 10;```.

6. Остановите проект. В качестве ответа приложите скриншот sql-запроса.

![docker](https://github.com/vadimtsvetkov/-virt-04-docker-in-practice/blob/main/screenshots/Screenshot_3.png)
