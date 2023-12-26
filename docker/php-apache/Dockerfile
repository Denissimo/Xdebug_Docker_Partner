FROM ubuntu:18.04
WORKDIR "/var/www/html"

ENV TZ=Europe/Moscow
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Packages
RUN apt-get update && \
    apt-get install -y \
        lbzip2 \
        gcc \
        autoconf \
        apache2 \
        apache2-dev \
        iputils-ping \
        mc \
        openssl \
        git \
        unzip \
        curl \
        mysql-client \
        libcurl4-gnutls-dev \
        build-essential \
        libbz2-dev \
        libreadline-dev \
        libsqlite3-dev \
        libssl-dev \
        libxml2-dev \
        libxslt1-dev \
        libpng-dev \
        libmcrypt-dev \
        php7.2-cli \
        php7.2-bz2 \
        php7.2-curl \
        php7.2-dev \
        pkg-config \
        memcached \
        libaio1

RUN ln -s /usr/include/x86_64-linux-gnu/curl /usr/local/include/curl

# # PHPbrew
RUN curl -L -O https://github.com/phpbrew/phpbrew/releases/download/1.27.0/phpbrew.phar \
    && chmod +x phpbrew.phar \
    && mv phpbrew.phar /usr/local/bin/phpbrew


RUN phpbrew init \
    && phpbrew update --old

RUN phpbrew install 5.3.29  +bz2 +calendar +cli +gd +ctype +dom +iconv +fileinfo +filter +ipc +json +mbregex +mbstring \
+mhash +mcrypt +pcntl +pcre +pdo +phar +posix +readline +sockets +tokenizer +xml +curl +zip +mysql +debug +apxs2 +mysql


# ORACLE oci 
RUN mkdir /opt/oracle \
    && cd /opt/oracle     
    
COPY oracle /opt/oracle

# Install Oracle Instantclient
RUN  unzip /opt/oracle/instantclient-basic-linux.x64-12.2.0.1.0.zip -d /opt/oracle \
    && unzip /opt/oracle/instantclient-sdk-linux.x64-12.2.0.1.0.zip -d /opt/oracle \
    && unzip /opt/oracle/instantclient-sqlplus-linux.x64-12.2.0.1.0.zip -d /opt/oracle \
    && ln -s /opt/oracle/instantclient_12_2/libclntsh.so.12.1 /opt/oracle/instantclient_12_2/libclntsh.so \
    && ln -s /opt/oracle/instantclient_12_2/libclntshcore.so.12.1 /opt/oracle/instantclient_12_2/libclntshcore.so \
    && ln -s /opt/oracle/instantclient_12_2/libocci.so.12.1 /opt/oracle/instantclient_12_2/libocci.so \
    && ln -s /opt/oracle/instantclient_12_2/sqlplus /usr/bin/sqlplus \
    && rm -rf /opt/oracle/*.zip \
    && mkdir /opt/oracle/instantclient_12_2/network \
    && mkdir /opt/oracle/instantclient_12_2/network/admin

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer --2.1
RUN composer --version

ENV LD_LIBRARY_PATH  /opt/oracle/instantclient_12_2:${LD_LIBRARY_PATH}
ENV PATH /opt/oracle/instantclient_12_2:$PATH
ENV TNS_ADMIN /opt/oracle/instantclient_12_2/sdk/admin
ENV NLS_LANG="AMERICAN_AMERICA.UTF8"
ENV NLS_DATE_FORMAT="DD.MM.YYYY"

SHELL ["/bin/bash", "-c"]
RUN ln -s /root/.phpbrew/php/php-5.3.29/bin/phpize /usr/bin/phpize5.3
RUN pecl config-set ext_dir /usr/lib/php/20151012 \
    && pecl config-set bin_dir /root/.phpbrew/php/php-5.3.29/bin/ \
    && pecl config-set php_bin /root/.phpbrew/php/php-5.3.29/bin/php \
    && pecl config-set php_ini /root/.phpbrew/php/php-5.3.29/etc/php.ini
RUN echo 'instantclient,/opt/oracle/instantclient_12_2' | pecl install -f oci8-2.0.12 \
    pecl install memcache-2.2.7 \
    pecl install timezonedb-2014.9

RUN source ~/.phpbrew/bashrc \
    && phpbrew use 5.3.29 \
    && phpbrew ext install xdebug 2.2.7 \
    && phpbrew ext enable oci8 \
    && phpbrew ext enable memcache \
    && phpbrew ext enable timezonedb

#RUN phpbrew ext install --pecl xdebug 2.2.7

RUN a2dismod mpm_event && a2enmod mpm_prefork && a2enmod ssl
EXPOSE 80 443
CMD apache2ctl -D FOREGROUND