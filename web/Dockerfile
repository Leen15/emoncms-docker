# Offical Docker PHP & Apache image https://hub.docker.com/_/php/
# PHP 8.0 is EOL: use a supported release (8.3 verified with core 11.6.9 and the pinned modules)
FROM php:8.3-apache

# Install deps
RUN apt-get update && apt-get install -y \
              libcurl4-gnutls-dev \
              libmcrypt-dev \
              gettext \
              nano \
              git-core \
              supervisor

# Enable PHP modules
RUN docker-php-ext-install -j$(nproc) mysqli gettext

# Install redis
COPY install_redis.sh /
RUN chmod +x /install_redis.sh
RUN /install_redis.sh

# Install mosquitto
COPY install_mosquitto.sh /
RUN chmod +x /install_mosquitto.sh
RUN /install_mosquitto.sh

RUN a2enmod rewrite

# Add custom PHP config
COPY config/php.ini /usr/local/etc/php/

# Add custom Apache config
COPY config/emoncms.conf /etc/apache2/sites-available/emoncms.conf
RUN a2dissite 000-default.conf
RUN a2ensite emoncms

# NOT USED ANYMORE - GIT CLONE INSTEAD
# Copy in emoncms files, files can be mounted from local FS for dev see docker-compose
# ADD ./emoncms /var/www/html

# The modules are pinned to the exact commits shipped in the leen15/emoncms:11.6.9 image.
# Cloning their master makes every rebuild pull newer code than the 11.6.9 core: e.g. the
# app module calls tr(), which the core does not define, so it fails with
# "Call to undefined function tr()". Bump these commits together with the core version.
# Clone in master Emoncms repo & modules - overwritten in development with local FS files
RUN mkdir /var/www/emoncms
RUN git clone -b '11.6.9' --single-branch https://github.com/emoncms/emoncms.git /var/www/emoncms
RUN git clone https://github.com/emoncms/dashboard.git /var/www/emoncms/Modules/dashboard \
 && git -C /var/www/emoncms/Modules/dashboard checkout 77967e0adcbd82ef16d05f5ef29d54d02c2d9440
RUN git clone https://github.com/emoncms/graph.git /var/www/emoncms/Modules/graph \
 && git -C /var/www/emoncms/Modules/graph checkout 9c28d68225466a26c37a80775d780d37a2ef9e3e
RUN git clone https://github.com/emoncms/app.git /var/www/emoncms/Modules/app \
 && git -C /var/www/emoncms/Modules/app checkout 224c9c3048253a842d0ac8953df3fe2a11219cbe
RUN git clone https://github.com/emoncms/device.git /var/www/emoncms/Modules/device \
 && git -C /var/www/emoncms/Modules/device checkout c44fd979c1390a39bfc5231bae762386cd8b54dd

COPY docker.settings.ini /var/www/emoncms/settings.ini

# Create folders & set permissions for feed-engine data folders (mounted as docker volumes in docker-compose)
RUN mkdir /var/opt/emoncms
RUN mkdir /var/opt/emoncms/phpfina
RUN mkdir /var/opt/emoncms/phptimeseries
RUN chown www-data:root /var/opt/emoncms/phpfina
RUN chown www-data:root /var/opt/emoncms/phptimeseries

# Create Emoncms logfile
RUN mkdir /var/log/emoncms
RUN touch /var/log/emoncms/emoncms.log
RUN chmod 666 /var/log/emoncms/emoncms.log

# To start Apache and emoncms_mqtt from supervisord
COPY config/supervisord.conf /etc/supervisor/supervisord.conf
ENTRYPOINT [ "/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/supervisord.conf" ]
