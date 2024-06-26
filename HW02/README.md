# Установка и настройка PostgteSQL в контейнере Docker

## Подготовка к выполнению ДЗ
Создана ВМ в VirtualBox с ОС Ubuntu 20.04, в которой установил Docker, docker-compose и создал каталог /var/lib/postgres.

## Выполнение ДЗ
1. развернуть контейнер с PostgreSQL 15 смонтировав в него /var/lib/postgresql
    * Создаем docker-сеть:
        
      ```sudo docker network create pg-net```
   
    * подключаем созданную сеть к контейнеру сервера Postgres и запускаем контейнер:
       
    ```sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15```
    
	<img src="/HW02/xxx/pg_server.PNG" alt="pg_server.png" /> 
	
2. разворачиваем контейнер с клиентом postgres и подключаемся к контейнеру с сервером
   
```sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres``` 

   <img src="/HW02/xxx/connect_server.PNG" alt="connect_server.png" />   
    
   * создаем таблицу со строками:
    
```CREATE TABLE test_tab (id SERIAL PRIMARY KEY, title TEXT);```
    
```INSERT INTO test_tab (id, title) VALUES (1, 'Строка 1');```
    
```INSERT INTO test_tab (id, title) VALUES (2, 'Строка 2');```
    
<img src="/HW02/xxx/test_tab.PNG" alt="test_tab.png" />
    
3. подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера   
    
	<img src="/HW02/xxx/connect_nout.PNG" alt="connect_nout.png" />

4. удалить контейнер с сервером
   
   <img src="/HW02/xxx/dell_serv.PNG" alt="dell_serv.png" />

5. создать контейнер заново
   
   ```sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15```
   
   <img src="/HW02/xxx/new_server.PNG" alt="new_server.png" />

6. подключаемся снова из контейнера с клиентом к контейнеру с сервером и проверяем что данные остались на месте
   
   ```sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres```
   
   <img src="/HW02/xxx/connect_server2.PNG" alt="connect_server2.png" />
