## Containerising Spring Boot + MySQL using Docker

If you're new to Docker then it might take some time to grasp the underlined concepts of the Docker technology and how it works but once you go through them, the building, running and deploying application becomes so much easier which we're going to learn in this particular post.

This tutorial is based on Spring Boot application that connects with MySQL database but that doesn't mean that this article won't help others because I assure you once you go through this tutorial you'll be able to deploy any Spring Boot application with MySQL database connectivity so let's get into it.

Before we start, let me give you a brief overview of Docker.

Docker is an OS-level virtualisation technology in which you run multiple containers(your individual project actually) from images(everything that requires to run an application in just one single unit) that makes the deploying process so much easier and also taking care of isolation between the various applications running on the same bare machine.

On the bottom, you have hardware on top of that you installed a host OS that provides a platform to run multiple applications and Docker Runtime is one of the applications that you can run on your host OS without creating a new instance which means you can directly access the host machine resources such as Memory, CPU etc which is not in the case of VMs where you have a separate kernel for your every instance with a separate RAM and CPU.

If you want to learn more about this technology then you can find good articles on this blog. Do check it out.

So let's start deploying our first application in the Docker environment. It's recommended to understand the containers, images, volumes, etc in case you're new to Docker before you can start this tutorial.

Technologies used in this project:

1. Java
2. Spring Boot
3. MySQL
4. Docker

Although there are some other technologies which are used in this project you don't have to worry about that.

What you should be knowing for this tutorial?

You do not need to know certain things just to deploy this simple application all you need is a couple of minutes of yourself.

The first step would be to clone this project from the GitHub to your local file system with the below command. First, go to your desired project folder path and hit the below command to clone the project and once you all clone this project, everything you need will be there in the folder, just go through them to understand what each file does in the deployment of containers.

```
git clone https://github.com/rohiet98/springboot-docker.git
```

We're going to deploy on the docker so forgot about the tomcat and localhost database because when you deploy the application on the docker then all the related applications also need to be deployed for eg: If a Spring application is dependent on the MySQL or MongoDB then you need to pull the images of these applications from the Docker Hub.

If you don't know about Docker Hub then it's a repository that is accessible to everyone even without logging into it and you can also store your own images thereafter creating an account, it's much similar like a GitHub, in Docker Hub you store the images and not containers. There is a lot more to learn in this but will see it later.

To check whether you've docker installed or not, hit the below command. There are many commands to test it out.

```
docker version // gives you all information 
docker -v 
docker --version
```

Note: If you're using the full name then you should use double hyphen but if you want to make it short then make the hyphen also a bit short by putting only single hyphen. This is not just limited to version but everywhere in the Docker.

If you've docker installed then it's time to create a docker image of your project and to create an image you need a special file which is Dockerfile. If you've cloned the project then you'll see that there is a file with such name in the classpath that has some instructions in it.

```
FROM openjdk:8-jdk-alpine 
ARG JAR_FILE=target/springboot.jar 
COPY ${JAR_FILE} springboot.jar 
ENTRYPOINT ["java","-jar","/springboot.jar"] 
EXPOSE 8080
```

FROM: It takes the openjdk images from the Docker hub with the 8-jdk-alpine as a tag. This is required because it's a java project and you need a kit to compile and build the Java project. If it's a Node JS application, you need Node JS image.

ARG: It's just to make it simple. Just think of creating a variable that takes the JAR file from the target folder in the classpath. Make sure you have JAR available before you start building the image.

COPY: This will make a copy of the JAR file and will store in the virtual file system. This will be consumed when you run this image as a container.

ENTRYPOINT: This is to make it executable and run it. It's similar to CMD command and will start your Spring Boot project.

EXPOSE: It's optional, if you need your application to be run on a different port, type it here.

Now it's time to build the project. All you have to do is first change the database credentials from the application.properties file according to yours and clean install your project once to get things reflected.

```
spring.datasource.url = jdbc:mysql://springboot-db:3306/USERS?useSSL=true&allowPublicKeyRetrieval=true
spring.datasource.username = rohit 
spring.datasource.password = 1234 \
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.MySQL5InnoDBDialect 
spring.jpa.hibernate.ddl-auto = update 
logging.level.org.hibernate.SQL=DEBUG 
logging.level.org.hibernate.type=TRACE
```

Here we've hardcoded everything but once you switch to Kubernetes from Docker, these hardcoded parameters will turn into loosely coupled parameters that will see it later. Here only one thing to keep in mind, the URL of the data source.

If you remember here then we've been using localhost in the URL which should not be the case when deploying to docker because docker containers usually don't interact with the localhost database because docker provides you with all kinds of containers which you need and MySQL is also one of them. Every container is known by its name in the network so you must provide the name of the container here rather than localhost because this is important. Just change it to your desired container name, in my case it is springboot-db.

Now everything seems to be right. Go the project root path and hit

```
mvn eclipse:eclipse && mvn clean install
```

Note: mvn eclipse command is optional. If you're working on the command line only, use mvn clean install only. With the above command, the jar file will be generated which is an input to the Dockerfile. The name of the jar must be the same as what you have provided in the Dockerfile, in my case, it's springboot.jar

To build a docker image, hit the below command

```
docker build -t springboot:1.0 .
```

Note: Don't forget to put the period in the end, it represents the entire files, folders available in the path that is going to be used in building a single image. Change the springboot to whatever name you want to give to your docker image, if not then let it be as it is. In the end, there is a tag that shows which version of the image you're building if you're dealing with so many versions of the same project then this can be of great use.

So you've built the image but with just a Spring Boot image, you won't be able to run the application, along with this you need the mysql image as well which can be pulled with the below command.

```
docker pull mysql:5.7
```

Note: When you pull any image, the docker first looks at the local file system if it's there then it won't pull anything else will pull from the registry. Order is first local, if not found, fetch it from the registry.

Now check how many images you have with the below command. You can use any of them and both are acceptable. This will list down all the images that are present on your machine along with the tag, id, size etc.

```
docker images ls docker image ls -a
```

You can manually run the container from the images one by one but we'll automating some of the things and docker-compose is one such tool that allows you to run multiple containers with just one command and for that, there is a requirement of one file that is YAML file to store the information about the images from which you're going to create containers.

In the clone project, you'll see one file named as docker-compose.yml

```
version: '3'
services:
    springboot:
        container_name: springboot
        image: 100598/springboot:1.0
        ports: 
        - 9090:8080
    mysql:
        container_name: mysql
        image: mysql:5.7
        environment:
        - MYSQL_ROOT_PASSWORD=root
        - MYSQL_USER=rohit
        - MYSQL_PASSWORD=1234
        - MYSQL_DATABASE=USERS
        volumes:
        - appvolume:/var/lib/mysql
volumes:
    appvolume:
```
    
Version is important, don't forget to add it. It's the version of docker-compose and not docker. Services are nothing but your containers which are going to be created when you execute this file. If you need to run 10 containers, put them all here along with some necessary information. In the end, we have used volumes. Usually, when you stop the container, all the data present on that container gets deleted and when you start the container again, it starts from a new state and that's not good when you are running an application with a stateful container such as a database container.

Just think, you have created an account of 10 users in your application and the moment you remove the container, the data gets lost. By using the volumes, you can persist the data. But how?

There are two file systems, physical and virtual. Your host has a physical filesystem while the containers have a virtual filesystem which is temporary. With the help of volumes, you can map the virtual filesystem to the physical filesystem so that all the data will be stored in the physical filesystem and thus making it persistent so you can use it every time you create a new container, data is always there.

Change the image name if you've a different name for your image. Also, the tag needs to be the same as well.

```
MYSQL_ROOT_PASSWORD=root // it should be root always 
MYSQL_USER=rohit // it will auto create the user 
MYSQL_PASSWORD=1234 // it will auto set the password for the above user 
MYSQL_DATABASE=USERS // it will auto create the database
```

Note: The root password should match with your own root password which is by default root. If you have modified then do changes here also.

The user will be automatically created and also the database but make sure that you use the name of environment variables correctly else you will get an error.

So now everything seems to be right even more. Now, it's time to turn up the switch. Here we go.

```
docker-compose up -d
```

This will create the containers automatically, first, it will read the AML file and will create the containers according to the data present in it. If you have a different file name of the docker-compose then do like this:

```
docker-compose -f docker-compose-new.yml up -d
```

Finally, check whether the containers are active or not. Hopefully, they're.

```
docker ps
```

Now hit the http://localhost:8080 in the browser and you will see the application is up. This is a Spring and MySQL application that helps only three simple pages, index, login and registration. Hope this article will help you.

The project is available on the GitHub: https://github.com/rohiet98/springboot-docker
