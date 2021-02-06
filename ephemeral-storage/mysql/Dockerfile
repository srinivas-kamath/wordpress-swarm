#We will be using mysql as the base image as it is required to store and run the database
FROM mysql:latest

#We will setup a non-interactive Environment to suppress any interactive activities during the image building process
ENV DEBIAN_FRONTEND=noninteractive

#Now, we will run the apt update and upgrade to update the existing packages and then run apt install for the required dependencies
RUN apt-get update -qq 2>/dev/null \
&& apt-get upgrade -qq 2>/dev/null \
&& apt-get autoremove -qq 2>/dev/null \
&& apt-get autoclean -qq 2>/dev/null \
&& apt-get install vim iputils-ping -qq 2>/dev/null \
&& rm -rf /var/lib/apt/lists/*

#Lastly, we run the command to start mysql service on container startup which will run as the database for our wordpress setup
CMD ["mysqld"]