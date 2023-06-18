FROM ubuntu:22.04

LABEL maintainer="Yousef <yousef.dev20@gmail.com>"

ARG WWWGROUP
ARG NODE_VERSION=18
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid

WORKDIR /var/www/html/magento

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update \
    && apt-get install -y gnupg gosu curl ca-certificates zip unzip git supervisor sqlite3 libcap2-bin libpng-dev python2 dnsutils nano net-tools \
    && curl -sS 'https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x14aa40ec0831756756d7f66c4f4ea0aae5267a6c' | gpg --dearmor | tee /etc/apt/keyrings/ppa_ondrej_php.gpg > /dev/null \
    && echo "deb [signed-by=/etc/apt/keyrings/ppa_ondrej_php.gpg] https://ppa.launchpadcontent.net/ondrej/php/ubuntu jammy main" > /etc/apt/sources.list.d/ppa_ondrej_php.list \
    && apt-get update \
    && apt-get install -y php7.4-cli php7.4-dev \
       php7.4-pgsql php7.4-sqlite3 php7.4-gd php7.4-imagick \
       php7.4-curl \
       php7.4-imap php7.4-mysql php7.4-mbstring \
       php7.4-xml php7.4-zip php7.4-bcmath php7.4-soap \
       php7.4-intl php7.4-readline \
       php7.4-ldap \
       php7.4-msgpack php7.4-igbinary php7.4-redis php7.4-swoole \
       php7.4-pcov php7.4-xdebug apache2 libapache2-mod-php7.4 \
    && curl -sLS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ --filename=composer \
    && curl -sLS https://deb.nodesource.com/setup_$NODE_VERSION.x | bash - \
    && apt-get install -y nodejs \
    && npm install -g npm \
    && apt-get update \
    && apt-get install -y mysql-client \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN setcap "cap_net_bind_service=+ep" /usr/bin/php7.4

# Update the PHP.ini file, enable <? ?> tags and quieten logging.
RUN sed -i "s/short_open_tag = Off/short_open_tag = On/" /etc/php/7.4/cli/php.ini
RUN sed -i "s/short_open_tag = Off/short_open_tag = On/" /etc/php/7.4/apache2/php.ini
RUN sed -i "s/allow_url_include = Off/allow_url_include = On/" /etc/php/7.4/cli/php.ini
RUN sed -i "s/allow_url_include = Off/allow_url_include = On/" /etc/php/7.4/apache2/php.ini
RUN sed -i "s/error_reporting = .*$/error_reporting = E_ERROR | E_WARNING | E_PARSE/" /etc/php/7.4/cli/php.ini
RUN sed -i "s/error_reporting = .*$/error_reporting = E_ERROR | E_WARNING | E_PARSE/" /etc/php/7.4/apache2/php.ini


RUN a2enmod php7.4
RUN a2enmod ssl
RUN a2enmod rewrite

COPY start-container /usr/local/bin/start-container
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY php.ini /etc/php/7.4/cli/conf.d/99-marketplace.ini
COPY 000-default.conf /etc/apache2/sites-enabled/000-default.conf

RUN chmod +x /usr/local/bin/start-container

EXPOSE 8000

ENTRYPOINT ["start-container"]
