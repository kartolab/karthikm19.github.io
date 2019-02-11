---
layout: post
title: "Creating Django project with MySQL using Docker"
tags: Docker Django Mysql
comments: true
---

A Step-by-step guide to create a Django project with MySQL using Docker.

<!-- more -->

### About Docker

Docker is a computer program that performs operating-system-level virtualization, also known as "containerization". This is a powerful tool for development because it makes the process of configuring and setting up environments or switching between the different projects in the same development box.

Read more about docker [here](https://www.docker.com).

### Prerequisites

As we discussed, we are going to use Docker, we will be using `Docker` and `Docker Compose`.

### Let's start

As an example we are going to create a Django application from scratch with MySQL.

Go to your project directory and create a sub-directory called `.docker`.

#### Requirements file

Instead of running too many `pip install` commands in `Dockerfile` we can list all the dependencies in a file and pass it to the install command so it will process each file during the docker build. 

- Create a file `.docker/requirements.txt`
- Copy the below content and save the file

  ```
  django==1.11
  mysqlclient==1.3.14
  ```

#### Dockerfile

Django is a Python web framework so we need to create a Python image and install Django on it.

- Create a file `Dockerfile`
- Copy the below content

```
FROM python:3.6

ENV PYTHONUNBUFFERED 1

# Enable the below proxy config if you are using any proxies 
# ENV http_proxy example-proxy.com:80
# ENV https_proxy example-proxy.com:80

RUN mkdir /application
WORKDIR "/application"

ADD requirements.txt /application/
RUN pip install -r requirements.txt
```

where,

Setting `PYTHONUNBUFFERED=1` allows for log messages to be immediately dumped to the stream instead of being buffered. This is useful for receiving timely log messages and avoiding situations where the application crashes without emitting a relevant message due to the message being "stuck" in a buffer.

Remove the comments `ENV http_proxy` and `ENV https_proxy` and add your proxy URLs if your machine is on proxy.

Adding the `requirements.txt` we have just created to the image.

`RUN pip install` passing the `requirements.txt` to install the dependencies.

#### Docker Compose

Docker Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application's services. Then, with a single command, you create and start all the services from your configuration.

- Go to project directory and create a file `docker-compose.yml`.
- Copy the below code

  ```
  version: "2"

  services:
      djangoweb:
        build: ./.docker
        command: python3 manage.py runserver 0.0.0.0:8000
        volumes:
          - .:/application
        ports:
          - "8000:8000"
        container_name: djangoweb
  ```

Go to terminal and run the below command

```
docker-compose build
```
