ARG PHP_VERSION
FROM ${PHP_VERSION}

ARG TZ
ARG PHP_EXTENSIONS
ARG CONTAINER_PACKAGE_URL

# 复制源
COPY sources.list /etc/apt/
COPY extensions/libzip-1.8.0.tar.gz /root/
COPY ./extensions/librdkafka-1.8.2.tar.gz /root/

# 安装基础组件
RUN apt-get update  \
    && apt-get install -y vim telnet curl wget zip unzip procps iputils-ping cmake  \
    && apt-get install -y libssl-dev libbz2-dev libmcrypt4 libmcrypt-dev libxslt1.1 libxslt1-dev  \
    libpng-dev libjpeg-dev libfreetype6-dev libevent-dev

# 手动编译
RUN cd /root && tar -zxvf libzip-1.8.0.tar.gz  \
    && cd libzip-1.8.0 && mkdir build  \
    && cd build && cmake .. && make && make install \
    && cd /root && tar -zxvf librdkafka-1.8.2.tar.gz  \
    && cd librdkafka-1.8.2 && ./configure && make && make install

#pecl安装
RUN pecl install nsq \
    && pecl install grpc  \
    && pecl install protobuf

# 创建php源目录
RUN docker-php-source extract

# 安装Redis扩展
# 将扩展解压并拷贝到源目录下
#ADD extensions/redis-5.3.5.tgz /usr/src/php/ext
#5eplay需要安装4.3.0版本
ADD extensions/redis-4.3.0.tgz /usr/src/php/ext
ADD extensions/mongodb-1.12.0.tgz /usr/src/php/ext
ADD extensions/rdkafka-5.0.2.tgz /usr/src/php/ext
ADD extensions/xdebug-2.9.1.tgz /usr/src/php/ext

# docker-php-ext-install将安装并启动扩展
RUN docker-php-ext-install sockets  \
    && docker-php-ext-install pcntl  \
    && docker-php-ext-install pdo_mysql  \
    && docker-php-ext-install gd  \
    && docker-php-ext-install bz2  \
    && docker-php-ext-install zip  \
    && docker-php-ext-install bcmath  \
    && docker-php-ext-install mbstring  \
    && docker-php-ext-install mysqli  \
    && docker-php-ext-install redis-4.3.0  \
    && docker-php-ext-install mongodb-1.12.0  \
    && docker-php-ext-install rdkafka-5.0.2  \
    && docker-php-ext-install xdebug-2.9.1  \
    && docker-php-ext-enable nsq \
    && docker-php-ext-enable grpc \
    && docker-php-ext-enable protobuf

# Install composer and change it's cache home
RUN curl -o /usr/bin/composer https://mirrors.aliyun.com/composer/composer.phar \
    && chmod +x /usr/bin/composer
ENV COMPOSER_HOME=/tmp/composer

# php image's www-data user uid & gid are 82, change them to 1000 (primary user)
RUN usermod -u 1000 www-data && groupmod -g 1000 www-data

WORKDIR /www
