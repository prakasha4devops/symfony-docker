# See https://github.com/docker-library/php/blob/4677ca134fe48d20c820a19becb99198824d78e3/7.0/fpm/Dockerfile
FROM php:7.1-fpm

MAINTAINER Prakash A

# Environmental Variables
ENV PHP_MEMORY_LIMIT 1024M
ENV PHP_DATE_TIMEZONE UTC

# Install additional packages
RUN apt-get update && apt-get install -y git openssh-client unzip wget curl build-essential --no-install-recommends

# Install PHP extensions
RUN buildDeps=" \
        libicu-dev \
        libpq-dev \
        libmcrypt-dev \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng12-dev \
        libbz2-dev \
        libxslt-dev \
        libcurl4-openssl-dev \
        libmemcached-dev \
        zlib1g-dev \
        libncurses5-dev \
        libc-client-dev \
        libkrb5-dev \
        libpspell-dev \
        libtidy-dev \
        libxml2 \
        libxml2-dev \
        git  \
        git-core  \
        vim  \
        nano  \
        # for mongodb
         libssl-dev \
        # for rabbit mq
         librabbitmq-dev \
         cron \
    " \
    && apt-get install -y $buildDeps --no-install-recommends \
    && rm -r /var/lib/apt/lists/* \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-configure pdo_mysql --with-pdo-mysql=mysqlnd \
    && docker-php-ext-install \
      gettext \
      exif \
      curl \
      calendar \
      intl \
      mbstring \
      mcrypt \
      pcntl \
      pdo \
      pdo_mysql \
      zip \
      opcache \
      iconv \
      mcrypt \
      gd \
      bcmath \
      bz2 \
      xsl \
      xmlrpc \
      mysqli \
      pspell \
      shmop \
      sockets \
      sysvmsg \
      sysvsem \
      sysvshm \
      tidy \
      json \
      ftp  \
      soap


# Install memcached
RUN apt-get install -y libmemcached-dev \
  && git clone https://github.com/php-memcached-dev/php-memcached /usr/src/php/ext/memcached \
  && cd /usr/src/php/ext/memcached && git checkout -b php7 origin/php7 \
  && docker-php-ext-configure memcached \
  && docker-php-ext-install memcached

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN composer --version

# Set timezone
RUN rm /etc/localtime
RUN ln -s /usr/share/zoneinfo/Europe/London /etc/localtime
RUN "date"

# Install php-imap
RUN docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install imap


# Install phpunit and put binary into $PATH
RUN curl -sSo phpunit.phar https://phar.phpunit.de/phpunit.phar \
    && chmod 755 phpunit.phar \
    && mv phpunit.phar /usr/local/bin/ \
    && ln -s /usr/local/bin/phpunit.phar /usr/local/bin/phpunit

# Install PHP Code sniffer
RUN curl -OL https://squizlabs.github.io/PHP_CodeSniffer/phpcs.phar \
    && chmod 755 phpcs.phar \
    && mv phpcs.phar /usr/local/bin/ \
    && ln -s /usr/local/bin/phpcs.phar /usr/local/bin/phpcs \
    && curl -OL https://squizlabs.github.io/PHP_CodeSniffer/phpcbf.phar \
    && chmod 755 phpcbf.phar \
    && mv phpcbf.phar /usr/local/bin/ \
    && ln -s /usr/local/bin/phpcbf.phar /usr/local/bin/phpcbf


# install mongodb ext
RUN pecl install mongodb \
    && docker-php-ext-enable mongodb

# Memory Limit
RUN echo "memory_limit=${PHP_MEMORY_LIMIT}" > /usr/local/etc/php/conf.d/docker-php-ext-memory-limit.ini

# Time Zone
RUN echo "date.timezone=${PHP_DATE_TIMEZONE}" > /usr/local/etc/php/conf.d/docker-php-ext-date-timezone.ini

# install sendmail package
RUN apt-get update && apt-get install -y sendmail sendmail-bin

# Setup PHP to use mailcatcher image to send mails
# The name of the linked container must be "mailcatcher"

# Install dependencies
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -q -y \
    libsqlite3-dev \
    ruby \
    ruby-dev \
    build-essential && \
    gem install --no-ri --no-rdoc mailcatcher --version 0.5.12 && \
    apt-get remove -y build-essential && \
    apt-get autoremove -y && \
    apt-get clean

# sendmail
RUN echo "SMTP=mailcatcher" > /usr/local/etc/php/conf.d/docker-php-ext-mailcatcher.ini
#RUN echo "smtp_port=25" > /usr/local/etc/php/conf.d/docker-php-ext-sendmail.ini
RUN echo "sendmail_path=/usr/bin/env catchmail --smtp-ip mailcatcher -f docker@symfony.dev" > /usr/local/etc/php/conf.d/docker-php-ext-mailcatcher.ini

#restart  sendmail if not started by default
RUN service sendmail restart

#RUN apt-get update &&  apt-get install -y  ssmtp && \
#    echo "sendmail_path = /usr/sbin/ssmtp -t" > /usr/local/etc/php/conf.d/sendmail.ini && \
#    echo "mailhub=mail:25\nUseTLS=NO\nFromLineOverride=YES" > /etc/ssmtp/ssmtp.conf

RUN sed -i -r 's/\;ping.path/ping.path/'  /usr/local/etc/php-fpm.d/www.conf \
    && sed  -r 's/\;ping.response = pong/ping.response=OK/' -i  /usr/local/etc/php-fpm.d/www.conf

# install xdebug
RUN pecl install xdebug
RUN docker-php-ext-enable xdebug
RUN echo "error_reporting = E_ALL" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "display_startup_errors = true" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "display_errors = true" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_enable=true" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_connect_back=false" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.idekey=\"PHPSTORM\"" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_port=9001" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_host=symfony.dev" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_autostart=true" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_handler=dbgp" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

RUN echo "xdebug.remote_log=/var/log/xdebug.log" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

RUN echo "opcache.load_comments=1" >> /usr/local/etc/php/conf.d/docker-php-ext-opcache.ini
RUN echo "opcache.save_comments = 1" >> /usr/local/etc/php/conf.d/docker-php-ext-opcache.ini


RUN echo 'alias sf="php /var/www/symfony/app/console"' >> ~/.bashrc
RUN echo 'alias sf3="php /var/www/symfony/bin/console"' >> ~/.bashrc

RUN echo 'alias sf-clear-cache="php /var/www/symfony/app/console cache:clear --env=dev && php /var/www/symfony/app/console cache:clear --env=prod"' >> ~/.bashrc
RUN echo 'alias sf3-clear-cache="php /var/www/symfony/bin/console cache:clear --env=dev && php /var/www/symfony/bin/console cache:clear --env=prod"' >> ~/.bashrc

RUN echo 'alias sf-clear-log="cat /dev/null > /var/www/symfony/app/logs/dev.log;cat /dev/null > /var/www/symfony/app/logs/prod.log;"' >> ~/.bashrc
RUN echo 'alias sf3-clear-log="cat /dev/null > /var/www/symfony/var/logs/dev.log;cat /dev/null > /var/www/symfony/var/logs/prod.log;"' >> ~/.bashrc

WORKDIR /var/www/symfony
