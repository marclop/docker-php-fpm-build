FROM debian:jessie

# persistent / runtime deps
RUN apt-get update && apt-get install --force-yes -y sudo git wget openssh-client libmysqld-dev libssl-dev libtidy-dev libmemcached-dev libmcrypt4 libjpeg62 ca-certificates libpng12-0 libxslt1.1 curl libpcre3 librecode0 libsqlite3-0 libxml2 --no-install-recommends && rm -r /var/lib/apt/lists/*

# phpize deps
RUN apt-get update && apt-get install --force-yes -y autoconf file g++ gcc libc-dev make pkg-config re2c --no-install-recommends && rm -r /var/lib/apt/lists/*

ENV PHP_INI_DIR /usr/local/etc/php
RUN mkdir -p $PHP_INI_DIR/conf.d

## Extra
ENV PACKAGE_LIST memcache tidy

# Apache
RUN apt-get update && apt-get install --force-yes -y apache2 apache2.2-common --no-install-recommends && rm -rf /var/lib/apt/lists/*

RUN rm -rf /var/www/html && mkdir -p /var/lock/apache2 /var/run/apache2 /var/log/apache2 /var/www/html && chown -R www-data:www-data /var/lock/apache2 /var/run/apache2 /var/log/apache2 /var/www/html

RUN mv /etc/apache2/apache2.conf /etc/apache2/apache2.conf.dist && rm /etc/apache2/sites-enabled/*
COPY apache2.conf /etc/apache2/apache2.conf
# it'd be nice if we could not COPY apache2.conf until the end of the Dockerfile, but its contents are checked by PHP during compilation

ENV PHP_EXTRA_BUILD_DEPS apache2-dev
ENV PHP_EXTRA_CONFIGURE_ARGS --with-apxs2=/usr/bin/apxs2

#ENV GPG_KEYS F38252826ACD957EF380D39F2F7956BC5DA04B5D
#RUN set -xe \
#	&& for key in $GPG_KEYS; do \
#		gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
#	done

# Copy Docker
COPY docker-php-ext-* /usr/local/bin/

# PHP 5.3
#ENV PHP_VERSION 5.4.43
ENV PHP_VERSION 5.6.22

## OPENSSL 0.9.8zg
#ENV OPENSSL_VERSION 0.9.8zb
#ENV OPENSSL_URL https://www.openssl.org/source/openssl-$OPENSSL_VERSION.tar.gz
#RUN curl -L -o /usr/src/openssl.tar.gz $OPENSSL_URL\
#    && cd /usr/src/ \
#    && tar xzf /usr/src/openssl.tar.gz \
#    && cd openssl-$OPENSSL_VERSION \
#    && ./config shared --openssldir=/usr/local/ssl \
#    && mkdir /usr/local/ssl \
#    && make -j"$(nproc)" \
#    && make -j"$(nproc)" install

# --enable-mysqlnd is included below because it's harder to compile after the fact the extensions are (since it's a plugin for several extensions, not an extension in itself)
RUN buildDeps=" \
$PHP_EXTRA_BUILD_DEPS \
libmhash2 \
libmcrypt-dev \
bzip2 \
unzip \
libbz2-dev \
libicu-dev \
libgmp3-dev \
libpcre3-dev \
libjpeg-dev \
libpng12-dev \
libreadline6-dev \
librecode-dev \
libsqlite3-dev \
libssl-dev \
libxml2-dev \
libcurl4-gnutls-dev \
libxslt1-dev \
libltdl-dev \
" \
	&& set -x \
	&& apt-get update && apt-get install --force-yes -y $buildDeps --no-install-recommends && rm -rf /var/lib/apt/lists/* \
	&& curl -SL "http://php.net/get/php-$PHP_VERSION.tar.bz2/from/this/mirror" -o php.tar.bz2 \
  && ln -s /usr/include/x86_64-linux-gnu/gmp.h /usr/include/gmp.h \
	#&& curl -SL "http://php.net/get/php-$PHP_VERSION.tar.bz2.asc/from/this/mirror" -o php.tar.bz2.asc \
	#&& gpg --verify php.tar.bz2.asc \
	&& mkdir -p /usr/src/php \
	&& tar -xof php.tar.bz2 -C /usr/src/php --strip-components=1 \
	&& rm php.tar.bz2* \
	&& cd /usr/src/php \
	&& ./configure \
		--with-config-file-path="$PHP_INI_DIR" \
		--with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
		$PHP_EXTRA_CONFIGURE_ARGS \
    --disable-cgi \
    --enable-bcmath \
    --enable-calendar \
    --enable-exif \
    --enable-mbstring \
		--enable-mysqlnd \
    --enable-pcntl \
    --enable-shmop \
    --enable-soap \
    --enable-sockets \
    --enable-sysvmsg \
    --enable-sysvsem \
    --enable-sysvshm \
    --enable-wddx \
    --enable-zip \
    --with-bz2 \
		--with-curl \
    --with-gettext \
    --with-gd \
    --with-gmp \
    --with-jpeg-dir \
		--with-mcrypt \
    --with-mysql \
    --with-mysqli \
    --with-openssl-dir=/usr/local/ssl \
    --with-openssl \
    --with-pcre-regex \
    --with-pdo-mysql \
		--with-readline \
		--with-recode \
		--with-zlib \
    --with-xsl \
    --with-tidy \
	&& make -j"$(nproc)" \
	&& make install \
	&& { find /usr/local/bin /usr/local/sbin -type f -executable -exec strip --strip-all '{}' + || true; } \
  && chmod +x /usr/local/bin/docker-php-ext-* && sync \
  && cd /usr/src/php/ext && pecl download $PACKAGE_LIST \
  && for PACKAGE in $PACKAGE_LIST; do tar zxf $PACKAGE-* \
  && rm -f $PACKAGE*.tgz && mv $PACKAGE-* $PACKAGE && docker-php-ext-configure $PACKAGE; done \
  && curl -LO https://s3-eu-west-1.amazonaws.com/cmp-docker-files/php-waencrypt.zip \
  && unzip php-waencrypt.zip && rm -f php-waencrypt.zip && mv php-waencrypt waencrypt \
  && pecl download memcached xdebug-2.2.7 && tar xzf memcached-* && rm -f memcached-*.tgz && mv memcached-* memcached \
  && tar xzf xdebug-* && rm -f xdebug-*.tgz && mv xdebug-* xdebug \
  && docker-php-ext-configure waencrypt \
  && docker-php-ext-configure memcached --disable-memcached-sasl \
  && docker-php-ext-install xdebug memcache tidy memcached waencrypt \
  && echo "zend_extension=\"/usr/local/lib/php/extensions/no-debug-zts-20090626/xdebug.so\"" > /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
  && docker-php-ext-configure waencrypt \
  && docker-php-ext-install waencrypt \
  && docker-php-ext-configure tidy \
  && docker-php-ext-install tidy && rm -f /usr/local/etc/php/conf.d/docker-php-ext-tidy.ini \
	&& apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false $buildDepss \
  && php -m

##<autogenerated>##
COPY apache2-foreground /usr/local/bin/
WORKDIR /var/www/html

EXPOSE 80
CMD ["apache2-foreground"]
##</autogenerated>##
