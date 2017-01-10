# Use phusion/baseimage as base image. To make your builds
# reproducible, make sure you lock down to a specific version, not
# to `latest`! See
# https://github.com/phusion/baseimage-docker/blob/master/Changelog.md
# for a list of version numbers.
FROM phusion/baseimage:0.9.18

MAINTAINER Tomasz Szymanek <tszymanek@o2.pl>

# Ensure UTF-8
RUN locale-gen pl_PL.UTF-8
ENV LANG       pl_PL.UTF-8
ENV LC_ALL     pl_PL.UTF-8

# Set correct environment variables.
ENV HOME /

RUN sed -i "s/^exit 101$/exit 0/" /usr/sbin/policy-rc.d

RUN rm -f /etc/service/sshd/down
# Regenerate SSH host keys. baseimage-docker does not contain any, so you
# have to do that yourself. You may also comment out this instruction; the
# init system will auto-generate one during boot.
# Use baseimage-docker's init system.
RUN /etc/my_init.d/00_regen_ssh_host_keys.sh

ENV TERM xterm

# Use baseimage-docker's init system.
CMD ["/sbin/my_init"]

RUN apt-get update && DEBIAN_FRONTEND="noninteractive" apt-get upgrade -y -o Dpkg::Options::="--force-confold"

RUN apt-get update && DEBIAN_FRONTEND="noninteractive" apt-get install -y \
	vim \
	curl \
	wget \
	build-essential \
	software-properties-common \
	python-software-properties \
	git \
	nano \
	acl

# Apache 2.4, PHP 7 and Installation
RUN add-apt-repository -y ppa:ondrej/apache2
RUN add-apt-repository -y ppa:ondrej/php
RUN apt-get update

# Apache 2.4
RUN DEBIAN_FRONTEND="noninteractive" apt-get install -y \
	apache2 \ 
	libapache2-mod-php7.1

RUN service apache2 stop
RUN a2enmod headers && a2enmod rewrite
# Apache + PHP requires preforking Apache for best results
RUN a2dismod mpm_event && a2enmod mpm_prefork

ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOCK_DIR /var/lock/apache2
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid
ENV APACHE_RUN_DIR /var/run/apache2

# PHP 7
RUN DEBIAN_FRONTEND="noninteractive" apt-get install -y \ 
	--force-yes \
	php7.1-cli \
	php7.1-common \
	php7.1-curl \
	php7.1-dev \
	php7.1-intl \
	php7.1-json \
	php7.1-mysql \
	php7.1-pgsql \
	php7.1-xml \
	php7.1-zip \
	php-pear \
	php-xdebug

# Xdebug
RUN sed -i "s/zend_extension=xdebug.so/zend_extension=\/usr\/lib\/php\/20160303\/xdebug.so/" /etc/php/7.1/mods-available/xdebug.ini \
	&& echo "xdebug.remote_enable=1\n \
			 xdebug.remote_port=9000\n \
			 xdebug.idekey=PHPSTORM\n \
			 xdebug.remote_connect_back = 1" \
			 >> /etc/php/7.1/mods-available/xdebug.ini

# Composer
ENV COMPOSER_HOME /var/www/.composer
RUN curl -sS https://getcomposer.org/installer | php -- \
	--install-dir=/usr/local/bin \
	--filename=composer

RUN mkdir -p $COMPOSER_HOME/cache \
    && chown 1000:1000 -R $COMPOSER_HOME

# MongoDB Driver
RUN pecl install mongodb \
    && echo "extension=mongo.so" > /etc/php/7.1/mods-available/mongo.ini

RUN mkdir -p 			/var/www/html

RUN mkdir				/etc/service/apache2
ADD build/symfony.conf	/etc/apache2/sites-available/symfony.conf
RUN a2dissite 000-default && a2ensite symfony
ADD build/apache2.sh	/etc/service/apache2/run
RUN chmod +x			/etc/service/apache2/run

ADD build/permissions.sh /etc/my_init.d/permissions.sh

RUN useradd -u 1000 docker && usermod -a -G root,www-data docker

VOLUME /var/www/html
WORKDIR /var/www/html
EXPOSE 80

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*