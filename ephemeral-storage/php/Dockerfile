#We will be using php:7.4-apache as the base image as it is required to store and run the php code along with apache bundled in it
FROM php:7.4.14-apache

#We will setup a non-interactive Environment to suppress any interactive activities during the image building process
ENV DEBIAN_FRONTEND=noninteractive

#Now, we will run the apt update and upgrade to update the existing packages and then run apt install for the required dependencies
RUN apt-get update -qq 2>/dev/null \
&& apt-get upgrade -qq 2>/dev/null \
&& apt-get autoremove -qq 2>/dev/null \
&& apt-get autoclean -qq 2>/dev/null \
&& apt-get install vim wget tree iputils-ping libfreetype6-dev libjpeg62-turbo-dev libpng-dev libmcrypt-dev -qq 2>/dev/null \
&& rm -rf /var/lib/apt/lists/* \
&& docker-php-ext-configure gd --with-freetype --with-jpeg \
&& docker-php-ext-install -j$(nproc) gd \
&& docker-php-ext-install mysqli \
&& pecl install mcrypt-1.0.3 \
&& docker-php-ext-enable mcrypt \
&& a2enmod rewrite \
&& rm -rf /etc/apache2/sites-enabled/000-default.conf

#Next, we add the customised apache conf file that detacts wordpress .htaccess file in the webroot directory and download the wordpress code zip and unzip it inside the container
COPY .htaccess .
COPY 000-default.conf /etc/apache2/sites-enabled/000-default.conf
RUN wget https://wordpress.org/latest.tar.gz \
&& tar xzf latest.tar.gz \
&& mv wordpress/* /var/www/html/ \
&& rm -rf /var/www/html/wordpress /var/www/html/latest.tar.gz \
&& chown -R www-data:www-data /var/www/html \
&& find /var/www/html -type d -exec chmod -R 755 {} \; \
&& find /var/www/html -type f -exec chmod -R 644 {} \;

#Lastly, we run the command to start apache service on container startup which will execute requests for php files proxied from nginx container and run as the code environment for our wordpress setup
CMD ["apache2-foreground"]