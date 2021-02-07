Sharing the commands to make this setup working in your system using docker-compose :-

First, download this repo :-

    $ mkdir /docker && cd /docker/ && git clone https://github.com/srinivas-kamath/wordpress-swarm.git

Next, we will make some changes to the docker-compose.yml file's environment part :-

    $ cd /docker/wordpress-swarm/ephemeral-storage/docker-compose/

Open the docker-compose.yml file in your preferred file-editor in CLI or GUI and update the "MYSQL_ROOT_PASSWORD" and "MYSQL_PASSWORD" variables in environment part of the file :-

FROM

    environment:
        - MYSQL_ROOT_PASSWORD=<random-strong-password>
        - MYSQL_DATABASE=wordpress
        - MYSQL_USER=wordpress-user
        - MYSQL_PASSWORD=<random-strong-password>

TO

    environment:
        - MYSQL_ROOT_PASSWORD=r2nu3JEDAQpKPE7EwpCxG8d8WtUoLV2c
        - MYSQL_DATABASE=wordpress
        - MYSQL_USER=wordpress-user
        - MYSQL_PASSWORD=D34qLyTkK2vQmzuJzZR8Z5uDaMVkcE7N

NOTE :- The Database passwords shared above in the "MYSQL_ROOT_PASSWORD" and "MYSQL_PASSWORD" variables are for reference only; the reader following the steps needs to create random strong passwords of their own and replace them with the passwords specified to make their container setup as secure as possible.

Once the above steps are done; the next part is to just run the following commands :-

    $ cd /docker/wordpress-swarm/ephemeral-storage/docker-compose/
    $ docker-compose up --detach

This last command will start building the images for the containers as well as launch all the 3 containers with all the required variables and configurations.

Once the containers are up and running, check your wordpress setup by hitting the Host-ip on 80 port which; "wordpress_nginx" container should accept and proxy the request to "wordpress_php" container and greet you with a wordpress-initial setup page.

On the wordpress-initial setup page; add the database name, username, password and database hostname for initial-setup as specified in the environment variable for "wordpress_mysql" and "wordpress_php" container and the wordpress installation should complete successfully and you should be able to see the default wordpress page on hitting the host-ip.
