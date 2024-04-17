# Установка и настройка PostgteSQL в контейнере Docker

## Подготовка к выполнению ДЗ
Создана ВМ в VirtualBox с ОС Ubuntu 20.04, в которой установил Docker, docker-compose и создал каталог /var/lib/postgres.

## Выполнение ДЗ
1. развернуть контейнер с PostgreSQL 15 смонтировав в него /var/lib/postgresql
    * Создаем docker-сеть:
        
      ```sudo docker network create pg-net```
   
    * подключаем созданную сеть к контейнеру сервера Postgres и запускаем контейнер:
       
    ```sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15```
    картинка 1
2. разворачиваем контейнер с клиентом postgres и подключаемся к контейнеру с сервером
   
```sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres```    
    
    * создаем таблицу со строками:
    
```CREATE TABLE test_tab (id SERIAL PRIMARY KEY, title TEXT);```
    
```INSERT INTO test_tab (id, title) VALUES (1, 'Строка 1');```
    
```INSERT INTO test_tab (id, title) VALUES (2, 'Строка 2');```
    картинка 3
    
4. подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера   
    
