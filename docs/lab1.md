### Контейниризация средствами Docker
Dockerfile для запуска бэкенд части проекта

```
FROM python:3.8.12

RUN apt-get update && apt-get upgrade -y && apt-get autoclean

RUN mkdir /project
COPY . /project/
WORKDIR /project

RUN pip install -r requirements.txt

EXPOSE 8000
CMD python manage.py runserver 0.0.0.0:8000
```
Dockerfile для запуска фронтенд части проекта

```
FROM node:lts-alpine

ENV PYTHONUNBUFFERED=1

WORKDIR /front

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 8080

CMD ["npm", "run", "serve"]

```
Оркестрация с помощью docker-compose

```
version: '3'

services:
        tours_db:
                container_name: pg_container
                image: postgres
                ports:
                        - "5433:5432"
                environment:
                        - POSTGRES_USER=postgres
                        - POSTGRES_PASSWORD=postgres
                        - POSTGRES_DB=tours_db
                volumes:
                        - ./dbs/postgres-data:/var/lib/postgresql
        pgadmin:
                container_name: pgadmin4_container
                image: dpage/pgadmin4
                environment:
                        PGADMIN_DEFAULT_EMAIL: admin@admin.com
                        PGADMIN_DEFAULT_PASSWORD: root
                ports:
                        - "5050:80"
                depends_on:
                        -  tours_db
        back:
                container_name: back
                build:
                        context: .
                        dockerfile: Dockerfile
                command: bash -c "python3 manage.py makemigrations && python3 manage.py migrate && python3 manage.py runserver 0.0.0.0:8000"
                ports:
                        - "8000:8000"
        front:
                container_name: tours_front
                build: ./front
                ports:
                        - "8080:8080"
                depends_on:
                        - back
```
После выполнения миграций с помощью
```
python manage.py makmigrations
python manage.py migrate
```

![](/images/migrations.png)

Результат оркестрации

![](/images/orch.png)