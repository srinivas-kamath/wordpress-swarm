To setup the containers to run in swarm mode; we will need to make some configurations prior to running the "docker service" commands like creating secrets and clearing out the trail of the procedure in this scenario :-

The Steps :-

First we; initiate the swarm mode in the existing system to make the setup launchable in swarm mode :-

        $ docker swarm init

If you are setting up swarm mode in a cloud server then you may receive an error :-

        # Error response from daemon: could not choose an IP address to advertise since this system has multiple addresses on different interfaces (xxx.xx.x.x on eth1 and xxx.xxx.x.xx on eth0) - specify one with --advertise-addr

In that case just choose the public-IP of that cloud server in the swarm enabling command because the Public-IP of the Cloud Server is how other remote servers will be able to connect and communicate; Please do remember this crucial point as if you add the <Private_IP-Address> by mistake then you may face connectivity issues between your different nodes :-

        $ docker swarm init --advertise-addr <Public_IP-Address>

You will receive an output with a string that contains details related to the swarm mode that you just initialized;

        $ docker swarm join --token <random-token> <Public_IP-Address>:2377

You just need to copy this single-line string and paste it into other remote systems that you want to add to this setup as remote nodes.

I am going to run 3 different remote nodes; 1 per container but you can make the changes as per your requirements.

Once you have run the string in all the remote nodes that you want to add to this setup; You can fire another command to check/confirm that all nodes are connected and you should get similar output if you are running a 3-node cluster :-

        $ docker node ls

        ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
        33xq9e56vz42etk496gtmylpj *   node1               Ready               Active              Leader              19.03.11
        hd8jspx78s4em8l6mnk5yedjq     node2               Ready               Active                                  19.03.11
        j6us2nj96vq5u2i14g5oevi6j     node3               Ready               Active                                  19.03.11

Now, We create 2 different overlay networks to isolate the mysql container from the web-server container without denying access to the php code container :-

        $ docker network create --driver overlay wordpress_frontend
        $ docker network create --driver overlay wordpress_backend

Now, we specify the secrets that we want to be used between the containers to be able to communicate and authenticate between each other :-

        $ echo "<root-user_password>" | docker secret create mysql_root_password -
        # e.g. :- echo "T9SL3C4JRAXV3EEPKSYAWNYUP6WNVDST" | docker secret create mysql_root_password -

        $ echo "<database-name>" | docker secret create mysql_database -
        # e.g. :- echo "wordpress" | docker secret create mysql_database -

        $ echo "<admin-username>" | docker secret create mysql_user -
        # e.g. :- echo "wordpress-user" | docker secret create mysql_user -

        $ echo "<admin-user_password>" | docker secret create mysql_password -
        # e.g. :- echo "T43ST8RRXBY8R844FALVY68QBREEZ7XU" | docker secret create mysql_password -

        $ echo "<mysql_container_hostname>" | docker secret create wordpress_db_host -
        # e.g. :- echo "wordpress_mysql" | docker secret create wordpress_db_host -

NOTE :- The Database passwords shared above in "e.g. :-" commands are for reference only;
        the reader following the steps needs to create random strong passwords of their own and replace them with the passwords specified in the "e.g. :-" commands to make their container setup as secure as possible.

Once the secrets are created you need to clear your user's command history as the secrets entered can be retrieved from bash history with a simple "history" command if not cleared :-

        $ history -c && history -w

Now; we launch the mysql container first in the swarm system :-

        $ docker service create --network wordpress_backend --name wordpress_mysql --secret mysql_root_password --secret mysql_database --secret mysql_user --secret mysql_password --env MYSQL_ROOT_PASSWORD_FILE=/run/secrets/mysql_root_password --env MYSQL_DATABASE_FILE=/run/secrets/mysql_database --env MYSQL_USER_FILE=/run/secrets/mysql_user --env MYSQL_PASSWORD_FILE=/run/secrets/mysql_password srinivaskamath/wordpress-swarm:wordpress_mysql

Once the command is entered you can check whether the container is runing via both swarm specific commands or with docker cli commands once you have figured out in which node the container has been launched by the swarm service.

To check in which node the container has been launched, simply enter :-

        $ docker service ls

        ID                  NAME                MODE                REPLICAS            IMAGE                                         PORTS
        ynp38amoskgn        wordpress_mysql     replicated          1/1                 srinivaskamath/wordpress-swarm:wordpress_mysql

This command's output confirms that the container has been launched.
Now, the next command is to find out in which node our container has been launched :-

        $ docker service ps wordpress_mysql

        ID                  NAME                IMAGE                                         NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
        kgpduyaq1wh0        wordpress_mysql.1   srinivaskamath/wordpress-swarm:wordpress_mysql   node1               Running             Running 8 minutes ago

From the output we can see that our container was launched in "node1".
Now, you can go to "Node1" and run the docker cli commands to check with the container specific commands :-

        $  docker container ps --all

        CONTAINER ID        IMAGE                                         COMMAND                  CREATED             STATUS              PORTS                 NAMES
        7bacfe80daa8        srinivaskamath/wordpress-swarm:wordpress_mysql   "docker-entrypoint.s…"   11 minutes ago      Up 11 minutes       3306/tcp, 33060/tcp   wordpress_mysql.1.kgpduyaq1wh0lz28ogn62vh4m

Once you have confirmed that the container is running; now you can launch php container as well :-

        $ docker service create --network wordpress_backend --network wordpress_frontend --name wordpress_php --secret wordpress_db_host --env WORDPRESS_DB_HOST_FILE=/run/secrets/wordpress_db_host srinivaskamath/wordpress-swarm:wordpress_php

Since, this is a 3 node cluster we will launch 3 replica containers of the "wordpress_nginx" image so that we get a nginx container on each node for the frontend.

        $ docker service create --network wordpress_frontend --name wordpress_nginx --replicas 3 --publish 80:80 srinivaskamath/wordpress-swarm:wordpress_nginx

Once these commands are executed you can run a single swarm command to check as to which containers are launched in which nodes :-
    
        $ docker service ps wordpress_mysql wordpress_php wordpress_nginx
    
        ID                  NAME                IMAGE                                         NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
        v34dd1px39sf        wordpress_nginx.1   srinivaskamath/wordpress-swarm:wordpress_nginx   node2               Running             Running 8 minutes ago                        
        50ys4ghijisu        wordpress_php.1     srinivaskamath/wordpress-swarm:wordpress_php     node3               Running             Running 9 minutes ago                        
        oi1i5dk8kxs6        wordpress_mysql.1   srinivaskamath/wordpress-swarm:wordpress_mysql   node2               Running             Running 10 minutes ago                       
        v1evs7p2j3kk        wordpress_nginx.2   srinivaskamath/wordpress-swarm:wordpress_nginx   node1               Running             Running 8 minutes ago                        
        46fygt55m654        wordpress_nginx.3   srinivaskamath/wordpress-swarm:wordpress_nginx   node3               Running             Running 8 minutes ago

Once you see similar output as above; its time to test whether your WordPress setup on docker swarm responds to you request or not.

The cool part of swarm is you dont need to find out the specific node on which "wordpress_nginx" is running and dont need to send 80-port requests to;

You can just randomly choose any Node from the one's in the swarm cluster and can hit on their public-IP's 80-port; and the swarm networking running in these nodes will route the request to the respective node on which the "wordpress_nginx" container is running on its own without any noticeable lag or latency issues;

You can also always find out the specific Node on which the "wordpress_nginx" container is running and hit that Node's Public-IP on the 80-port and that will also work but I just explained the first-way to showcase swarm service's flexibility.