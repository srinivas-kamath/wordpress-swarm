Sharing the commands to make this setup working in your system :-

First, download this repo into your Downloads directory 

        $ cd /home/`whoami`/Downloads/ && git clone https://github.com/srinivas-kamath/wordpress-swarm.git

Next, we will build the images from the dockerfiles shared in the sub-folders :-

        $ cd /home/`whoami`/Downloads/wordpress-swarm/ephemeral-storage/mysql/
        $ sudo docker build --no-cache --tag wordpress_mysql .

        $ cd /home/`whoami`/Downloads/wordpress-swarm/ephemeral-storage/php/
        $ sudo docker build --no-cache --tag wordpress_php .

        $ cd /home/`whoami`/Downloads/wordpress-swarm/ephemeral-storage/nginx/
        $ sudo docker build --no-cache --tag wordpress_nginx .

Next, we will create 2 networks so that we can launch the containers in different networks to isolate the backend container from frontend containers :-

        $ sudo docker network create wordpress_backend
        $ sudo docker network create wordpress_frontend

Now, we launch the database container by using the image that we built from the dockerfile and attach it to the backend network and also add environment variables that specify the root user's password, the default database name, another database user's name and password for that new database user :-

        $ sudo docker container run --detach --network wordpress_backend --env MYSQL_ROOT_PASSWORD=<random-strong-password> --env MYSQL_DATABASE=wordpress --env MYSQL_USER=wordpress-user --env MYSQL_PASSWORD=<random-strong-password> --name wordpress_mysql wordpress_mysql

Next, we launch the php container which contains all the wordpress code and add an environment variable that sets the mysql container's hostname as the database hostname for the php container :-

        $ sudo docker container run --detach --network wordpress_backend --env WORDPRESS_DB_HOST=wordpress_mysql --name wordpress_php wordpress_php

The manual docker command to launch the php container cannot connect to more than 1 network at a time; so we will connect the newly launched php container to the frontend network manually so that it will be able to communicate with the web-server container which we are going to launch as the last step :-

        $ sudo docker network connect wordpress_frontend wordpress_php

Next, we launch the nginx container which will work as the frontend web server and so we attach it to the frontend network and also open the 80 port for this container so that the web requests that are sent to your system's 80 port gets routed to the nginx container's 80 port :-

        $ sudo docker container run --detach --network wordpress_frontend --publish 80:80 --name wordpress_nginx wordpress_nginx

So; the way that your requests will be routed now are :-

![Request Route](https://github.com/srinivas-kamath/wordpress-swarm/blob/main/ephemeral-storage/images/request-route.png)

Once the containers are up and running check your wordpress setup by hitting the Host-ip on 80 port which; "wordpress_nginx" container should accept and proxy the request to "wordpress_php" container and greet you with a wordpress-initial setup page.

On the wordpress-initial setup page; add the database name, username, password and database hostname for initial-setup as specified in the environment variable for "wordpress_mysql" and "wordpress_php" container and the wordpress installation should complete successfully and you should be able to see the default wordpress page on hitting the host-ip.