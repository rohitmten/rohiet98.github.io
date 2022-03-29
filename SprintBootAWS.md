##Deploying Spring Boot + MySQL on AWS EC2 using Docker

In this particular, I am going to explain you how you can run a Spring Boot application with MySQL database connection on Amazon EC2 Instance using Docker, you can run the same thing with Tomcat or Kubernetes or anything that opens at specific port. In this post, I will deploy my application using Docker and will access it using the AWS EC2 public IP.

Amazon Web Services (AWS) is the world's most comprehensive and broadly adopted cloud platform, offering over 175 fully featured services from data centers globally.

Amazon Elastic Compute Cloud is a part of Amazon.com's cloud-computing platform, Amazon Web Services, that allows users to rent virtual computers on which to run their own computer applications.

Read this complete post and by the end of this tutorial, you will be able to deploy your first application on EC2 Instance, so let's start.

Here is the outline

Pull the project from GitHub repository
Generate Docker Image from project by first generating the JAR/WAR file
Pushing the Docker Image on Docker Hub
Logging into AWS Console and setting up the instance
Installing Docker/Docker-Compose and other tools on the instance
Pulling Docker Image from Docker Hub
Running both the Spring Boot and MySQL container using Docker-Compose
Setting Port on AWS EC2 instance
App will be working...
These are the steps that we're going to follow so first would be pulling the project from the GitHub repository using the below clone command.

git clone https://www.github.com/rowhiet/mypriori
## Localhost Database Configuration
spring.datasource.url = jdbc:mysql://db:3306/MYPRIORI?useSSL=false&allowPublicKeyRetrieval=true
spring.datasource.username = root
spring.datasource.password = root
In the database source URL, you've to mention the name of the MySQL container so that the Spring Boot container can connect to the database as you have to omit the localhost here. After modifying the database credentials if you need, you have to generate the JAR file for the project using the below command.

mvn clean package
This will generate the JAR file in the target folder which will be input for your Dockerfile. Now build the Docker Image from the project using the below command.

docker build -t {docker-hub-username}/mypriori-app:1.0 .
Note: You've to use your Docker Hub username instead of {docker-hub-username}.

Now you have to push this image on to the Docker Hub because the EC2 instance can't take the image from your local machine. EC2 instance will take both the above generated image and mysql image from the Docker Hub when we run the project.

To push on to the Docker Hub, use this command:

docker push {docker-hub-username}/mypriori-app:1.0
Note: Replace {docker-hub-username} with your Docker Hub username.

Now, it's time to move on the virtual machine that is AWS EC2 instance. Follow these steps to set up the instance.

Login into AWS account
Click on Services and then EC2 under Compute
Click on Instances(Running)
Click on Launch Instances
Select the Machine Image from the list(Choose Amazon Linux 2 AMI)
Click on Review and Launch(Select free tier)
Click on Launch
Create your new key pair and download it on your machine
Now select the instance and click on the connect and follow the instruction
Navigate to the keypair file where you have downloaded
Hit the ssh command
You'll be entered into the remote shell of your EC2 instance and here you need to download everything as it's a new machine so install Docker, Maven, Docker-Compose and also other required applications.

Now clone the project from the GitHub URL in the EC2 instance terminal.

git clone https://www.github.com/rowhiet/mypriori
Now pull the Spring Boot project from Docker Hub and also the MySQL Image using below commands

docker pull {docker-hub-username}/mypriori-app.1:0
docker pull mysql:5.7
Now navigate to the project in the EC2 terminal and run the docker-compose file using the below command

docker-compose up -d
This will run both the Spring Boot and MySQL container and you can check them using docker ps command. The Spring Boot application runs on the 8080 port hence you have to add the 8080 port in your EC2 instance security groups.

Go the Inbound rules
Edit inbound rules
Add a new rule
Select type as Custom TCP and set protocol as TCP
Select port range as 8080
Select source as custom
Save rules
That's it, your application is now ready to be accessed. Copy the Public IPv4 DNS and open this URL

Note: Add the respective port at the end of URL, for this put :8080 after the public URL

I hope this tutorial helped you in setting up the first Spring Boot application with database connection on AWS using Docker.

You can find this project on GitHub: https://www.github.com/rowhiet/mypriori

