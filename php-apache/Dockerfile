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
ENV HOME /root

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
	libapache2-mod-php7.0

RUN service apache2 stop
RUN a2enmod headers && a2enmod rewrite
# Apache + PHP requires preforking Apache for best results
RUN a2dismod mpm_event && a2enmod mpm_prefork

ENV APACHE_RUN_USER docker
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOCK_DIR /var/lock/apache2
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid
ENV APACHE_RUN_DIR /var/run/apache2

# PHP 7
RUN DEBIAN_FRONTEND="noninteractive" apt-get install -y \ 
	--force-yes \
	php7.0-cli \
	php7.0-common \
	php7.0-curl \
	php7.0-dev \
	php7.0-intl \
	php7.0-json \
	php7.0-mysql \
	php7.0-xml \
	php7.0-zip \
	php-pear \
	php-xdebug

# Xdebug
RUN sed -i "s/zend_extension=xdebug.so/zend_extension=\/usr\/lib\/php\/20151012\/xdebug.so/" /etc/php/7.0/mods-available/xdebug.ini
RUN echo "xdebug.remote_enable=1\n\
xdebug.remote_port=9000\n\
xdebug.idekey=PHPSTORM\n\
xdebug.remote_connect_back = 1"\
 >> /etc/php/7.0/mods-available/xdebug.ini

# Composer
RUN curl -sS https://getcomposer.org/installer | php -- --filename=/usr/local/bin/composer

RUN mkdir -p 			/var/www/html

RUN mkdir				/etc/service/apache2
ADD build/symfony.conf	/etc/apache2/sites-available/symfony.conf
RUN a2dissite 000-default && a2ensite symfony
ADD build/apache2.sh	/etc/service/apache2/run
RUN chmod +x			/etc/service/apache2/run

ADD build/permissions.sh /etc/my_init.d/permissions.sh

RUN useradd -r -u 1000 -G www-data,root docker

VOLUME /var/www/html
WORKDIR /var/www/html
EXPOSE 80

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*