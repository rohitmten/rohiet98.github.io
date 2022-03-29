## Deploying Spring Boot + MySQL on AWS EC2 using Docker

In this particular, I am going to explain you how you can run a Spring Boot application with MySQL database connection on Amazon EC2 Instance using Docker, you can run the same thing with Tomcat or Kubernetes or anything that opens at specific port. In this post, I will deploy my application using Docker and will access it using the AWS EC2 public IP.

Amazon Web Services (AWS) is the world's most comprehensive and broadly adopted cloud platform, offering over 175 fully featured services from data centers globally.

Amazon Elastic Compute Cloud is a part of Amazon.com's cloud-computing platform, Amazon Web Services, that allows users to rent virtual computers on which to run their own computer applications.

Read this complete post and by the end of this tutorial, you will be able to deploy your first application on EC2 Instance, so let's start.

Here is the outline

1. Pull the project from GitHub repository
2. Generate Docker Image from project by first generating the JAR/WAR file
3. Pushing the Docker Image on Docker Hub
4. Logging into AWS Console and setting up the instance
5. Installing Docker/Docker-Compose and other tools on the instance
6. Pulling Docker Image from Docker Hub
7. Running both the Spring Boot and MySQL container using Docker-Compose
8. Setting Port on AWS EC2 instance
9. App will be working...

These are the steps that we're going to follow so first would be pulling the project from the GitHub repository using the below clone command.

```
git clone https://www.github.com/rohiet98/mypriori
```

```
## Localhost Database Configuration
spring.datasource.url = jdbc:mysql://db:3306/MYPRIORI?useSSL=false&allowPublicKeyRetrieval=true
spring.datasource.username = root
spring.datasource.password = root
```

In the database source URL, you've to mention the name of the MySQL container so that the Spring Boot container can connect to the database as you have to omit the localhost here. After modifying the database credentials if you need, you have to generate the JAR file for the project using the below command.

```
mvn clean package
```

This will generate the JAR file in the target folder which will be input for your Dockerfile. Now build the Docker Image from the project using the below command.

```
docker build -t {docker-hub-username}/mypriori-app:1.0 .
```

Note: You've to use your Docker Hub username instead of {docker-hub-username}.

Now you have to push this image on to the Docker Hub because the EC2 instance can't take the image from your local machine. EC2 instance will take both the above generated image and mysql image from the Docker Hub when we run the project.

To push on to the Docker Hub, use this command:

```
docker push {docker-hub-username}/mypriori-app:1.0
```

Note: Replace {docker-hub-username} with your Docker Hub username.

Now, it's time to move on the virtual machine that is AWS EC2 instance. Follow these steps to set up the instance.

1. Login into AWS account
2. Click on Services and then EC2 under Compute
3. Click on Instances(Running)
4. Click on Launch Instances
5. Select the Machine Image from the list(Choose Amazon Linux 2 AMI)
6. Click on Review and Launch(Select free tier)
7. Click on Launch
8. Create your new key pair and download it on your machine
9. Now select the instance and click on the connect and follow the instruction
10. Navigate to the keypair file where you have downloaded
11. Hit the ssh command

You'll be entered into the remote shell of your EC2 instance and here you need to download everything as it's a new machine so install Docker, Maven, Docker-Compose and also other required applications.

Now clone the project from the GitHub URL in the EC2 instance terminal.

```
git clone https://www.github.com/rohiet98/mypriori
```

Now pull the Spring Boot project from Docker Hub and also the MySQL Image using below commands

```
docker pull {docker-hub-username}/mypriori-app.1:0
docker pull mysql:5.7
```

Now navigate to the project in the EC2 terminal and run the docker-compose file using the below command

```
docker-compose up -d
```

This will run both the Spring Boot and MySQL container and you can check them using docker ps command. The Spring Boot application runs on the 8080 port hence you have to add the 8080 port in your EC2 instance security groups.

1. Go the Inbound rules
2. Edit inbound rules
3. Add a new rule
4. Select type as Custom TCP and set protocol as TCP
6. Select port range as 8080
7. Select source as custom
8. Save rules

That's it, your application is now ready to be accessed. Copy the Public IPv4 DNS and open this URL

Note: Add the respective port at the end of URL, for this put :8080 after the public URL

I hope this tutorial helped you in setting up the first Spring Boot application with database connection on AWS using Docker.

You can find this project on GitHub: https://www.github.com/rohiet98/mypriori

