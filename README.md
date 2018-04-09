##run on your local mach
	
	$ mvn clean package exec:java

##run on docker
	docker-compose up -d
	
##create new person
curl -H "Content-Type: application/json" -X POST -d "{\"first\": \"Mustafa\",\"last\": \"KOÇ\",\"dateofbirth\": 381110400000,\"placeofbirth\": \"Erzincan\"}" "http://192.168.99.100:8080/people"

##list existing people in the database
curl -H "Content-Type: application/json" -X GET "http://192.168.99.100:8080/people"


In this article, I am going to show you how to develop and run a simple Spring web application using Java 8 without installing Java 8 on your local machine.

Python developers use virtual environments for creating and managing separate environments for different projects, and each using different versions of Python for execution and storing and resolving Python dependencies. Java and many other technologies do not support a virtual environment concept. At this point, Docker comes to our aid.

Docker is a virtualization platform. I am not going dive into details of docker's details. you can find basic information and installation guide from the docker official site.

Once you have the Docker toolbox installed, you do not need install Java 8 or MySQL which are needed in our sample application.

Now, you can download my codes from GitHub.

First, let's check the Docker-compose file:

version : '2'
services:
  springappserver:
    build:
      context: . 
      dockerfile: springapp.dockerfile
    ports: 
      - "8080:8080"
    networks:
      - net-spring-db
    volumes:
      - .:/vol/development
    depends_on:
      - mysqldbserver
  mysqldbserver:
    build:
      context: . 
      dockerfile: mysqldb.dockerfile
    ports:
      - "3306:3306"
    networks:
      - net-spring-db
    environment:
      MYSQL_DATABASE: testdb
      MYSQL_USER: myuser
      MYSQL_PASSWORD: mypassword
      MYSQL_ROOT_PASSWORD: myrootpassword
    container_name: mysqldbserver
networks:
  net-spring-db:
    driver: bridge


We have two servers each on the 'net-spring-db' network. The first one is named 'springappserver' and configured with the springapp.dockerfile, which will be described later. The second one is named as mysqldbserver and configured with the mysqldb.dockerfile, which will be described later.

Now, let's have a look at the springapp.dockerfile:

#
# Java 1.8 & Maven Dockerfile
#
#
# pull base image.
FROM java:8
# maintainer
MAINTAINER Dursun KOC "dursunkoc@gmail.com"
# update packages and install maven
RUN  \
  export DEBIAN_FRONTEND=noninteractive && \
  sed -i 's/# \(.*multiverse$\)/\1/g' /etc/apt/sources.list && \
  apt-get update && \
  apt-get -y upgrade && \
  apt-get install -y vim wget curl maven
# attach volumes
VOLUME /vol/development
# create working directory
RUN mkdir -p /vol/development
WORKDIR /vol/development
# maven exec
CMD ["mvn", "clean", "package", "exec:java"]


This Docker file configures a Docker image, which is inherited from a Java 8 image from Docker Hub. Over that Java 8 image, I have installed vim, wget, curl, Maven, and set the volume in order to put my existing projects code. And finally, execute the Maven command to run my application.

Now let's check the mysqldb.dockerfile:

FROM mysql/mysql-server
MAINTAINER Dursun KOC <dursunkoc@gmail.com>
# Copy the database initialize script: 
# Contents of /docker-entrypoint-initdb.d are run on mysqld startup
ADD  mysql/ /docker-entrypoint-initdb.d/


This Dockerfile configures a Docker image, which is inherited from the MySQL/mysql-server image from Docker Hub. Over the MySQL image, I put my db-schema creation scripts, which are located in the MySQL folder. I have a single SQL file at this folder — data.sql — in order to create the 'person' table.

Now, let's see application structure.

Our application is started from the src/com/turkcell/softlab/Application.java file, and our only Controller is the PersonController(src/com/turkcell/softlab/controller/PersonController.java).

You can run the whole project with a simple command:

docker-compose up -d
For testing, use the following two commands in your local machine:

Create new person:

curl -H "Content-Type: application/json" -X POST -d "{\"first\": \"Mustafa\",\"last\": \"KOÇ\",\"dateofbirth\": 381110400000,\"placeofbirth\": \"Erzincan\"}" "http://192.168.99.100:8080/people"


List existing people in the database:

curl -H "Content-Type: application/json" -X GET "http://192.168.99.100:8080/people"


Now, it's your turn! You can dive deeper into Java 8 and Spring Boot using this template.
