---
layout: post
title: "How to install and use Composer in Docker"
tags: Docker PHP Composer
comments: true
---

Simple approach to install and use Composer in Docker container.
<!-- more -->
![Docker Django Mysql Banner](/assets/posts/docker-django-mysql-banner.png)

### Overview
[Docker](https://www.docker.com) is a computer program that performs operating-system-level virtualization, also known as "containerization".
[Composer](https://getcomposer.org/) is an application-level package manager for the PHP programming language that provides a standard format for managing dependencies of PHP software and required libraries.

### Let's start

### Composer Installer
Before I go into the details of creating `Dockerfile` and `Docker Compose` setup, I would to explain how a `Composer` installer works. 

According to the [installation steps](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-macos) in the Composer's documentation page, below is the command to install locally.

```
php composer-setup.php --install-dir=bin --filename=composer
```

#### Installer Options
`composer-setup.php` - the installer `https://getcomposer.org/installer`
`--install-dir`  - providing a target directory.
`--filename` - You can specify the filename (default: composer.phar)

### Install Composer in Docker image

#### Docker Compose
Docker Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application's services. Then, with a single command, you create and start all the services from your configuration.

- Go to project directory and create a new file called `docker-compose.yml`
- Copy the below below code
    ```
    version: '2'
    
    services:
      app:
        ports:
          - 9000:80
        build:
          context: .
        volumes:
          - .:/var/www/html
        container_name: exampleapp
    ```

#### Dockerifle
- Create a file called `Dockerfile` in the project directory.
- Copy the below code
    ```
    FROM php:7.3.0-apache

    # Enable the below lines if you are on any proxies with your proxy details
    #ENV http_proxy http://proxy.example.com:80
    #ENV https_proxy http://proxy.example.com:80
    
    RUN apt-get update && apt-get install -y git libzip-dev unzip \
        && docker-php-ext-install zip \
        && a2enmod rewrite headers
    
    RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
    
    WORKDIR /var/www/html
    ```
    
#### Create Image
In the above steps we have created the `Docker Compose` and `Dockerfile` that are required for Docker to create the image with Composer installed on it, and create a container using this image.

Now it's time to create the container. Go to terminal and build the image

<pre style="background-color: black; color: white;">
docker-compose build
</pre>

#### Create Container
Now we have an image to create as many container as we require. For our example I will create one container.

Go to terminal and run the below simple command.
```
docker-compose up -d
```
##### Output
<pre style="background-color: black; color: white;">
Starting exampleapp ... done
</pre>

#### Use Composer
We have just created container "exampleapp" in the above steps. Now let's go inside the container and see whether Composer is installed and available for us to use.

Go to terminal and run the below command to `bash` into the container
```
docker exec -it exampleapp bash
```
You will now see a prompt something like below
<pre style="background-color: black; color: white;">
root@2817d6672dca:/var/www/html# 
</pre>

Now run the below command to check the Composer version.
<pre style="background-color: black; color: white;">
root@2817d6672dca:/var/www/html/app# composer -v
</pre>

##### Output
The above command outputs the composer version, options and available commands on the screen, which means that composer is installed and available in container to use.
<pre style="background-color: black; color: white;">
Do not run Composer as root/super user! See https://getcomposer.org/root for details
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 1.8.4 2019-02-11 10:52:10

Usage:
  command [options] [arguments]

Options:
  -h, --help                     Display this help message
  -q, --quiet                    Do not output any message
  -V, --version                  Display this application version
      --ansi                     Force ANSI output
      --no-ansi                  Disable ANSI output
  -n, --no-interaction           Do not ask any interactive question
      --profile                  Display timing and memory usage information
      --no-plugins               Whether to disable plugins.
  -d, --working-dir=WORKING-DIR  If specified, use the given directory as working directory.
  -v|vv|vvv, --verbose           Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug
</pre>

Hope this article was useful for you.

Happy coding :)
