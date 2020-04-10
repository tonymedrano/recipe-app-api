# recipe-app-api
Recipe app api source code

#### Install
```shell script
- docker
```

#### Create and add to Dockerfile script
```shell script
FROM python:3.7-alpine
MAINTAINER Tony Medrano Developer

ENV PYTHONUNBUFFERED 1

COPY ./requirements.txt /requirements.txt
RUN pip install -r /requirements.txt

RUN mkdir /app
WORKDIR /app
COPY ./app /app

RUN adduser -D user
USER user
```
#### Run Dockerfile for creation
```shell script
sudo chmod 666 /var/run/docker.sock
docker build .
```

#### Create docker-compose.yml script
```shell script
version: "3.7"
services:
  app:
    build:
      context: .
    ports:
      - "8000:8000"
    volumes:
      - ./app:/app
    command: >
      sh -c "python manage.py runserver 0.0.0.0:8000
```
#### Run docker-compose.yml
```shell script
docker-compose build
```

#### Start new project
```shell script
docker-compose run app sh -c "django-admin.py startproject app ."
```