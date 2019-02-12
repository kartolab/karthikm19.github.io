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
##### output:
<pre style="background-color: black; color: white;">
Building djangoweb
Step 1/8 : FROM python:3.6
 ---> 5b0503f864f4
Step 2/8 : ENV PYTHONUNBUFFERED 1
 ---> Using cache
 ---> fb7e28e047ff
Step 3/8 : ENV http_proxy proxy.intra.bt.com:80
 ---> Using cache
 ---> 04eca146ce18
Step 4/8 : ENV https_proxy proxy.intra.bt.com:80
 ---> Using cache
 ---> d49277b01098
Step 5/8 : RUN mkdir /application
 ---> Using cache
 ---> 1434463762ea
Step 6/8 : WORKDIR "/application"
 ---> Using cache
 ---> 68fece209b83
Step 7/8 : ADD requirements.txt /application/
 ---> Using cache
 ---> fbc5ad46ccb5
Step 8/8 : RUN pip install -r requirements.txt
 ---> Using cache
 ---> 3e4a8a50a721
Successfully built 3e4a8a50a721
Successfully tagged dockerdjangomysql_djangoweb:latest
</pre>

#### Create a new Django project

Run the below command to create a new Django project `example`.

```
docker-compose run djangoweb django-admin startproject example .
```

If everything goes well, you will ge the prompt without any errors or messages.

`ls -ltr`

You will notice a new directory `example` and a python file `manage.py` has been generated in your project root directory.

Hurray, now it's time to run our Django example app. 

Go to your terminal and run the below command,

```
docker-compose up
```

It output something like below:

<pre style="background-color: black; color: white;">
Starting djangoweb
Attaching to djangoweb
djangoweb    | Performing system checks...
djangoweb    | 
djangoweb    | System check identified no issues (0 silenced).
djangoweb    | 
djangoweb    | You have 13 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
djangoweb    | Run 'python manage.py migrate' to apply them.
djangoweb    | February 11, 2019 - 09:57:04
djangoweb    | Django version 1.11, using settings 'example.settings'
djangoweb    | Starting development server at http://0.0.0.0:8000/
djangoweb    | Quit the server with CONTROL-C.
</pre>

where you can see that the server has been started successfully at `http://0.0.0.0:8000/`.

Open your preferred browser and go to [http://127.0.0.1:8000/](http://127.0.0.1:8000/) and you will see Django's welcome screen as shown below. 

![Welcome to Django](/assets/posts/welcome-to-django.png)

Excellent! 

Let's go ahead and setup MySQL for our project.

### Setup MySQL

#### Update `docker-compose`

We need to choose a MySQL image from docker hub for us to use in our setup. For our example I have chosen `MySQL 5.6`.

- Create a new directory `db-data` in your project root.

- Open `docker-compose.yml`

- Add the below config in the end.

  ```
  djangodb:
  image: mysql:5.6
  volumes:
  - ./db-data:/db-data
  environment:
  - MYSQL_DATABASE=docker_django_mysql
  - MYSQL_PORT=3306
  - MYSQL_USER=user
  - MYSQL_PASSWORD=pass
  - MYSQL_ROOT_PASSWORD=pass
  ports:
  - "8306:3306"
  container_name: djangodb
  ```

	where,

	  image is Mysql 5.6 as we discussed
      volume is to use the db-data to store database persistent data
      environment is used to create a new database for our example project on post 3306 with default username and password.

#### Run `docker-compose`

It is now time to build MySQL image in our machine. Go to terminal and run the below command.

```
docker-compose up
```

This will pull mysql image from Docker Hub if it doesn't already exists. Else it will justs start the container for `djangodb` and create the database with users we have configured.

If everything goes well then you will see both `djangoweb` and `djangodb` containers created successfully and in "Up" state by running the below command in a new terminal.

```
docker-compose ps
```
##### Output
<pre style="background-color: black; color: white;">
  Name                 Command               State           Ports          
---------------------------------------------------------------------------
djangodb    docker-entrypoint.sh mysqld      Up      0.0.0.0:8306->3306/tcp 
djangoweb   python3 manage.py runserve ...   Up      0.0.0.0:8000->8000/tcp 
</pre>

#### How to login to Mysql Database?

We have just created a Mysql container and so we should be able to login to Mysql and check whether everything has been created successfully.

Go to new terminal and run the below command,

```
docker-compose exec djangodb bash
```

which will open the container in an interactive mode with bash prompt.

Run the below command now to login to mysql

```
mysql -uroot -ppass
```
where, root/pass is the one we have configure in our `docker-compose.yml` under `djangodb`.

You will now see the `mysql` prompt where you can run any basic Mysql commands as shown below.

<pre style="background-color: black; color: white;">
mysql> show databases;
+---------------------+
| Database            |
+---------------------+
| information_schema  |
| docker_django_mysql |
| mysql               |
| performance_schema  |
+---------------------+
</pre>

where you can see the database "docker_django_mysql" has been created. Cool, isn't it?

### Connecting MySQL from Django

So, now we have both Django and Mysql container up and running. Below are the changes we have to make in Django application to use MySQL database.

Open `example/settings.py` and find the DATABASES config as shown below

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

By default Django creates an app with Sqlite3 database. In our example we are trying to use Mysql as database so we need tweak this config slightly as below.

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'HOST': 'djangodb',
        'NAME': 'docker_django_mysql',
        'USER': 'root',
        'PASSWORD': 'pass',
        'PORT': '3306'
    },
}
```

where ,
	
- ENGINE - 'django.db.backends.mysql' is the MYSQL engine for Django
- HOST - Mysql container. In our example we created 'djangodb'. Please refer `docker-compose.yml`
- NAME - Database name
- Credentials - Username and password
- PORT - port is the container's port

Go to terminal and `Ctrl-C` to cancel the `docker-compose up` and re-run the command again.

```
docker-compose up
```

Now you will probably getting error message like below in terminal. <span style="background-color:yellow; color:red;">Ignore this error for now.</span>

<pre style="background-color: black; color: white;">
djangoweb    |   File "/usr/local/lib/python3.6/site-packages/django/db/backends/mysql/base.py", line 274, in get_new_connection
djangoweb    |     conn = Database.connect(**conn_params)
djangoweb    |   File "/usr/local/lib/python3.6/site-packages/MySQLdb/__init__.py", line 85, in Connect
djangoweb    |     return Connection(*args, **kwargs)
djangoweb    |   File "/usr/local/lib/python3.6/site-packages/MySQLdb/connections.py", line 208, in __init__
djangoweb    |     super(Connection, self).__init__(*args, **kwargs2)
djangoweb    | django.db.utils.OperationalError: (2006, 'Can\'t connect to MySQL server on \'djangodb\' (111 "Connection refused")')
</pre>

This error is because `djangoweb` container has been created immediately and trying to connect to the database before `djangodb` has been started and ready to accept connections.

So, after good 20-30 seconds, open a new terminal and run the below command to restart only `djangoweb` container so that web container will be able to connect to the database now.

```
docker-compose restart djangoweb
```
Now, you will be able to see something like below:

<pre style="background-color: black; color: white;">
djangoweb    | You have 13 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
djangoweb    | Run 'python manage.py migrate' to apply them.
djangoweb    | February 12, 2019 - 17:00:52
djangoweb    | Django version 1.11, using settings 'example.settings'
djangoweb    | Starting development server at http://0.0.0.0:8000/
djangoweb    | Quit the server with CONTROL-C.
</pre>

Now all working fine so we don't really need to run the containers in foreground.

- Go to terminal and `ctrl-c` to close the container and run
	```
	docker-compose up -d
	```
	which will run the containers in detached mode.
	After few minutes, run
	```
	docker-compose restart djangoweb
	```
	to restart the web container to connect to databases. <span style="background-color:yellow; color:red;">This can be handled using a different mechanism by writing a bash script to wait and connect. I will probably do this in another post.</span>
	

### Database migrations

Now if you `bash` into the Mysql container, you will see a new table `django_migrations`.

In the above section you may notice the below message "<span style="color:maroon;">You have 13 unapplied migration(s).......</span>". This is to do the initial migration of tables and users. If you look at the next line of the above message there is a command Django is recommending for us to run,

Go to a new terminal and go inside the `djangoweb` container

```
docker-compose exec djangoweb bash
```
and run the command and you will see the migration process as shown below.

<pre style="background-color: black; color: white;">
root@372dbc6f2047:/application# python manage.py migrate
System check identified some issues:

WARNINGS:
?: (mysql.W002) MySQL Strict Mode is not set for database connection 'default'
	HINT: MySQL's Strict Mode fixes many data integrity problems in MySQL, such as data truncation upon insertion, by escalating warnings into errors. It is strongly recommended you activate it. See: https://docs.djangoproject.com/en/1.11/ref/databases/#mysql-sql-mode
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying sessions.0001_initial... OK
</pre>

If you now login to Mysql and you will see all the migrated tables now.

<pre style="background-color: black; color: white;">
mysql> show tables;
+-------------------------------+
| Tables_in_docker_django_mysql |
+-------------------------------+
| auth_group                    |
| auth_group_permissions        |
| auth_permission               |
| auth_user                     |
| auth_user_groups              |
| auth_user_user_permissions    |
| django_admin_log              |
| django_content_type           |
| django_migrations             |
| django_session                |
+-------------------------------+
10 rows in set (0.00 sec)
</pre>

#### Create `superuser`

There is no users setup at the moment so we need to setup a setup user to access the admin app.

- In terminal `bash` in to `djangoweb` container
- Run the below command and enter the required name, email and password as shown below:
	```
    root@372dbc6f2047:/application# python manage.py createsuperuser
    Username (leave blank to use 'root'): admin
    Email address: admin@eight19.github.io
    Password: 
    Password (again): 
    Superuser created successfully.
    ```

### Login to Admin app

Now we have done all the necessary steps to login to our admin app.

- Open browser and go to [http://127.0.0.1:8000/admin](http://127.0.0.1:8000/admin)
- Login using the admin username and password we have just created above and you will see admin home screen as below where you are manage the groups and users for your site.

![Django Admin Page](/assets/posts/django-admin-page.png)


Whoop whoop, you have successfully created and seutp a Django app.

Hope you have enjoyed this and you find this really useful.

Happy coding :)
