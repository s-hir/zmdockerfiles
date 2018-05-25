FROM ubuntu:16.04
MAINTAINER Suki Hirata

#
# Using el7 Dockerfile as template
# Eventually convert to docker compose
#

# Install LAMP
RUN apt-get update && apt-get upgrade -y && apt-get dist-upgrade -y \
  && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    apache2 \
    mariadb-server \
    php7.0 \
    php7.0-bcmath \
    php7.0-mcrypt \
    php7.0-mysql \
    libapache2-mod-php \
    zip

# fix up mysql install
RUN rm /etc/mysql/my.cnf
RUN cp /etc/mysql/mariadb.conf.d/50-server.cnf /etc/mysql/my.cnf

# Install the latest *release* of zoneminder
RUN apt-get install software-properties-common -y
RUN add-apt-repository ppa:iconnor/zoneminder
RUN apt-get update
RUN apt-get install zoneminder -y
RUN adduser www-data video
RUN a2enmod cgi
RUN a2enmod ssl
RUN a2ensite default-ssl
RUN a2enconf zoneminder
RUN a2enmod rewrite
RUN chown -R www-data:www-data /usr/share/zoneminder


# Initialize the database server
#RUN mysql_install_db --user=mysql --ldata=/var/lib/mysql/
#RUN mysql_secure_installation

# Set our volumes before we attempt to configure apache
VOLUME /var/lib/zoneminder/images /var/lib/zoneminder/events /var/lib/mysql /var/log/zoneminder

# Configure Apache
RUN mkdir -p /etc/httpd/conf.d
RUN echo "ServerName localhost" > /etc/httpd/conf.d/servername.conf
RUN echo -e "# Redirect the webroot to /zm\nRedirectMatch permanent ^/$ /zm" > /etc/httpd/conf.d/redirect.conf

# Configure timezone
ENV TZ=America/New_York
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN echo 'date.timezone = $TZ' >>  /etc/php/7.0/apache2/php.ini

# Expose https port
EXPOSE 443

# Get the entrypoint script and make sure it is executable
ADD https://raw.githubusercontent.com/ZoneMinder/zmdockerfiles/master/utils/entrypoint.sh /usr/local/bin/
RUN chmod 755 /usr/local/bin/entrypoint.sh

# This is run each time the container is started
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]

################
# RUN EXAMPLES #
################

# ZoneMinder uses /dev/shm for shared memory and many users will need to increase
# the size significantly at runtime like so:
#
# docker run -d -t -p 1080:443 \
#    --shm-size="512m" \
#    --name zoneminder \
#    zoneminder/zoneminder

# ZoneMinder checks the TZ environment variable at runtime to determine the timezone.
# If this variable is not set, then ZoneMinder will default to UTC.
# Alternaitvely, the timezone can be set manually like so:
#
# docker run -d -t -p 1080:443 \
#    -e TZ='America/Los_Angeles' \
#    --name zoneminder \
#    zoneminder/zoneminder

# ZoneMinder can write its data to folders outside the container using volumes.
#
# docker run -d -t -p 1080:443 \
#    -v /disk/zoneminder/events:/var/lib/zoneminder/events \
#    -v /disk/zoneminder/images:/var/lib/zoneminder/images \
#    -v /disk/zoneminder/mysql:/var/lib/mysql \
#    -v /disk/zoneminder/logs:/var/log/zm \
#    --name zoneminder \
#    zoneminder/zoneminder

# ZoneMinder can use an external database by setting the appropriate environment variables.
#
# docker run -d -t -p 1080:443 \
#    -e ZM_DB_USER='zmuser' \
#    -e ZM_DB_PASS='zmpassword' \
#    -e ZM_DB_NAME='zoneminder_database' \
#    -e ZM_DB_HOST='my_central_db_server' \
#    -v /disk/zoneminder/events:/var/lib/zoneminder/events \
#    -v /disk/zoneminder/images:/var/lib/zoneminder/images \
#    -v /disk/zoneminder/logs:/var/log/zm \
#    --name zoneminder \
#    zoneminder/zoneminder

# Here is an example using the options described above with the internal database:
#
# docker run -d -t -p 1080:443 \
#    -e TZ='America/Los_Angeles' \
#    -v /disk/zoneminder/events:/var/lib/zoneminder/events \
#    -v /disk/zoneminder/images:/var/lib/zoneminder/images \
#    -v /disk/zoneminder/mysql:/var/lib/mysql \
#    -v /disk/zoneminder/logs:/var/log/zm \
#    --shm-size="512m" \
#    --name zoneminder \
#    zoneminder/zoneminder

# Here is an example using the options described above with an external database:
#
# docker run -d -t -p 1080:443 \
#    -e TZ='America/Los_Angeles' \
#    -e ZM_DB_USER='zmuser' \
#    -e ZM_DB_PASS='zmpassword' \
#    -e ZM_DB_NAME='zoneminder_database' \
#    -e ZM_DB_HOST='my_central_db_server' \
#    -v /disk/zoneminder/events:/var/lib/zoneminder/events \
#    -v /disk/zoneminder/images:/var/lib/zoneminder/images \
#    -v /disk/zoneminder/logs:/var/log/zm \
#    --shm-size="512m" \
#    --name zoneminder \
#    zoneminder/zoneminder

