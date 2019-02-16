---
layout: post
title: "How to install and use Composer in Docker"
tags: Docker PHP Composer
comments: true
---

Simple approach to install and use Composer in Docker container.
<!-- more -->

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
    ```
- Go to terminal and build the image
<pre style="background-color: black; color: white;">
docker-compose build
</pre>

To be continued....
