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

# Install python mysql client
RUN apt-get update \
    && apt-get -y install libmysqlclient-dev \
    && apt-get clean; rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/*

ADD requirements.txt /application/
RUN pip install -r requirements.txt
```


