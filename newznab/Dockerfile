FROM ubuntu:14.04


ENV SVN_USERNAME svnusername
ENV SVN_PASSWORD svnpassword
ENV MYSQL_ROOT_PASSWORD dbpass
ENV php_timezone Europe/Paris 
ENV NEWS_PROVIDER_URL reader.usenetbucket.com
ENV NEWS_PROVIDER_LOGIN nntpusername
ENV NEWS_PROVIDER_PASSWORD nntppassword
ENV NEWS_PROVIDER_PORT 443
ENV NEWS_PROVIDER_SSL true

# Set correct environment variables
ENV DEBIAN_FRONTEND=noninteractive HOME="/root" TERM=xterm LANG=en_US.UTF-8 LANGUAGE=en_US:en LC_ALL=en_US.UTF-8

#Ports
EXPOSE 3306 
EXPOSE 80

# volumes
VOLUME /var/lib/mysql/ /var/www/newznab/nzbfiles/ /var/www/newznab/covers/

RUN locale-gen en_US.UTF-8

RUN echo "mysql-server-5.5 mysql-server/root_password password $MYSQL_ROOT_PASSWORD" | debconf-set-selections
RUN echo "mysql-server-5.5 mysql-server/root_password_again password $MYSQL_ROOT_PASSWORD" | debconf-set-selections

RUN apt-get update
RUN apt-get install -y --force-yes subversion supervisor php5 php5-curl apache2 openssl php-pear php5-mysql php5-curl php5-gd mysql-server-5.5 unrar-free libav-tools mediainfo

# MySQL
#RUN sed -i "s/max_allowed_packet      = 16M/max_allowed_packet      = 12582912/" /etc/mysql/my.cnf 

# PHP
RUN sed -i "s/max_execution_time = 30/max_execution_time = 120/" /etc/php5/cli/php.ini  && \
sed -i "s/memory_limit = -1/memory_limit = 1024M/" /etc/php5/cli/php.ini  && \
echo "register_globals = Off" >> /etc/php5/cli/php.ini  && \
echo "date.timezone =$php_timezone" >> /etc/php5/cli/php.ini  && \
sed -i "s/max_execution_time = 30/max_execution_time = 120/" /etc/php5/apache2/php.ini  && \
sed -i "s/memory_limit = -1/memory_limit = 1024M/" /etc/php5/apache2/php.ini  && \
echo "register_globals = Off" >> /etc/php5/apache2/php.ini  && \
echo "date.timezone =$php_timezone" >> /etc/php5/apache2/php.ini  && \
sed -i "s/memory_limit = 128M/memory_limit = 1024M/" /etc/php5/apache2/php.ini

# Apache2
RUN a2enmod rewrite
ADD src/newznab.apache /etc/apache2/sites-available/newznab.conf
RUN a2dissite 000-default.conf
RUN a2ensite newznab


# NewzNab
RUN svn co --username $SVN_USERNAME --password $SVN_PASSWORD svn://svn.newznab.com/nn/branches/nnplus /var/www/newznab

ADD src/config.php /var/www/newznab/www/config.php

RUN chmod 777 /var/www/newznab/www/lib/smarty/templates_c && \
chmod 777 /var/www/newznab/www/covers/movies && \
chmod 777 /var/www/newznab/www/covers/anime  && \
chmod 777 /var/www/newznab/www/covers/music  && \
chmod 777 /var/www/newznab/www/covers/tv  && \
chmod 777 /var/www/newznab/www  && \
chmod 777 /var/www/newznab/www/install  && \
chmod 777 /var/www/newznab/nzbfiles/ && \
chmod 777 /var/www/newznab/www/config.php

RUN sed -i "s/define('DB_PASSWORD','xxxxxxx');/define('DB_PASSWORD','$MYSQL_ROOT_PASSWORD');/" /var/www/newznab/www/config.php
RUN sed -i "s/define('NNTP_USERNAME', 'xxxxxxx');/define('NNTP_USERNAME', '$NEWS_PROVIDER_LOGIN');/" /var/www/newznab/www/config.php
RUN sed -i "s/define('NNTP_PASSWORD', 'xxxxxxx');/define('NNTP_PASSWORD', '$NEWS_PROVIDER_PASSWORD');/" /var/www/newznab/www/config.php
RUN sed -i "s/define('NNTP_SERVER', 'reader.usenetbucket.com');/define('NNTP_SERVER', '$NEWS_PROVIDER_URL');/" /var/www/newznab/www/config.php
RUN sed -i "s/define('NNTP_PORT', '443');/define('NNTP_PORT', '$NEWS_PROVIDER_PORT');/" /var/www/newznab/www/config.php
RUN sed -i "s/define('NNTP_SSLENABLED', true);/define('NNTP_SSLENABLED', $NEWS_PROVIDER_SSL);/" /var/www/newznab/www/config.php

COPY src/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

ADD src/newznab.sh /newznab.sh
ADD src/start.sh /usr/local/bin/start.sh
RUN chmod 755 /*.sh /usr/local/bin/start.sh

# Resatrting services
RUN service mysql restart && service apache2 restart

CMD ["/usr/bin/supervisord", "--nodaemon"]
CMD ["service", "mysql", "start"]
CMD ["service", "apache2", "start"]
CMD ["/usr/local/bin/start.sh"]


#Cleaning
RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
/usr/share/man /usr/share/groff /usr/share/info \
/usr/share/lintian /usr/share/linda /var/cache/man && \
(( find /usr/share/doc -depth -type f ! -name copyright|xargs rm || true )) && \
(( find /usr/share/doc -empty|xargs rmdir || true ))
