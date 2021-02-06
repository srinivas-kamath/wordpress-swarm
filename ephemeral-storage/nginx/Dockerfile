#We will be using nginx as the base image as it is required to catch and proxy all requests to the backend php container
FROM nginx:latest

#We will setup a non-interactive Environment to suppress any interactive activities during the image building process
ENV DEBIAN_FRONTEND=noninteractive

#Now, we will run the apt update and upgrade to update the existing packages and then run apt install for the required dependencies
RUN apt-get update -qq 2>/dev/null \
&& apt-get upgrade -qq 2>/dev/null \
&& apt-get autoremove -qq 2>/dev/null \
&& apt-get autoclean -qq 2>/dev/null \
&& apt-get install vim iputils-ping -qq 2>/dev/null \
&& rm -rf /var/lib/apt/lists/* \
&& rm -rf /etc/nginx/conf.d/default.conf

#Next, we add the custom default.conf created that proxies all requests to wordpress_php container
COPY default.conf /etc/nginx/conf.d/default.conf

#Lastly, we run the command to start nginx service on container startup which will run as the proxy-server for our wordpress setup
CMD ["nginx", "-g", "daemon off;"]