# See https://github.com/docker-library/php/blob/4677ca134fe48d20c820a19becb99198824d78e3/7.0/fpm/Dockerfile
FROM php:5.6-fpm

MAINTAINER Prakash Admane <prakash.admane@gmail.com>

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
      ftp


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


# install xdebug
RUN pecl install xdebug
RUN docker-php-ext-enable xdebug
RUN echo "error_reporting = E_ALL" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "display_startup_errors = On" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "display_errors = On" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_connect_back=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.idekey=\"PHPSTORM\"" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_port=9001" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_host=\"symfony.dev\"" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini



RUN echo 'alias sf="php app/console"' >> ~/.bashrc
RUN echo 'alias sf3="php bin/console"' >> ~/.bashrc

RUN echo 'alias sf-log-clear="cat /dev/null > app/logs/dev.log;cat /dev/null > app/logs/prod.log;"' >> ~/.bashrc
RUN echo 'alias sf3-log-clear="cat /dev/null > var/logs/dev.log;cat /dev/null > var/logs/prod.log;"' >> ~/.bashrc

WORKDIR /var/www/symfony