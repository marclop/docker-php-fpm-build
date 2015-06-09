FROM debian:jessie

# persistent / runtime deps
RUN apt-get update && apt-get install -y sudo git wget openssh-client libssl-dev libicu-dev ca-certificates curl libpcre3 librecode0 libsqlite3-0 libxml2 libxslt1.1 --no-install-recommends && rm -r /var/lib/apt/lists/*

# phpize deps
RUN apt-get update && apt-get install -y autoconf file g++ gcc libc-dev make pkg-config re2c --no-install-recommends && rm -r /var/lib/apt/lists/*

ENV PHP_INI_DIR /usr/local/etc/php
RUN mkdir -p $PHP_INI_DIR/conf.d

ENV PHP_EXTRA_CONFIGURE_ARGS --enable-fpm --with-fpm-user=www-data --with-fpm-group=www-data
ENV PACKAGE_LIST xdebug

ENV GPG_KEYS 6E4F6AB321FDC07F2C332E3AC2BF0BC433CFC8B3 0BD78B5F97500D450838F95DFE857D9A90D90EC1
RUN set -xe \
	&& for key in $GPG_KEYS; do \
		gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
	done

ENV PHP_VERSION 5.6.9

# --enable-mysqlnd is included below because it's harder to compile after the fact the extensions are (since it's a plugin for several extensions, not an extension in itself)
RUN buildDeps=" \
		$PHP_EXTRA_BUILD_DEPS \
    libmhash2 libmcrypt-dev \
		bzip2 \
    libbz2-dev \
    libgmp-dev \
		libpcre3-dev \
		libreadline6-dev \
		librecode-dev \
		libsqlite3-dev \
		libssl-dev \
		libxml2-dev \
    libcurl4-gnutls-dev \
    libxslt1-dev \
	" \
	&& set -x \
	&& apt-get update && apt-get install -y $buildDeps --no-install-recommends && rm -rf /var/lib/apt/lists/* \
	&& curl -SL "http://php.net/get/php-$PHP_VERSION.tar.bz2/from/this/mirror" -o php.tar.bz2 \
	&& curl -SL "http://php.net/get/php-$PHP_VERSION.tar.bz2.asc/from/this/mirror" -o php.tar.bz2.asc \
	&& gpg --verify php.tar.bz2.asc \
	&& mkdir -p /usr/src/php \
	&& tar -xof php.tar.bz2 -C /usr/src/php --strip-components=1 \
	&& rm php.tar.bz2* \
	&& cd /usr/src/php \
  && ln -s /usr/include/x86_64-linux-gnu/gmp.h /usr/include/gmp.h \
	&& ./configure \
		--with-config-file-path="$PHP_INI_DIR" \
		--with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
		$PHP_EXTRA_CONFIGURE_ARGS \
		--disable-cgi \
    --enable-bcmath \
    --enable-calendar \
    --enable-exif \
    --enable-intl \
    --enable-mbstring \
		--enable-mysqlnd \
    --enable-opcache \
    --enable-pcntl \
    --enable-shmop \
    --enable-sockets \
    --enable-wddx \
    --enable-zip \
    --with-bz2 \
		--with-curl \
    --with-gettext \
    --with-gmp \
    --with-mysql \
    --with-mysqli \
		--with-openssl \
    --with-pdo-mysql \
		--with-readline \
		--with-recode \
		--with-zlib \
    --with-xsl \
	&& make -j"$(nproc)" \
	&& make install \
	&& { find /usr/local/bin /usr/local/sbin -type f -executable -exec strip --strip-all '{}' + || true; } \
	&& apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false $buildDeps \
	&& make clean \
	&& ln -s $(which php) /usr/bin/php

COPY docker-php-ext-* /usr/local/bin/

# Download source for external PHP extensions
RUN cd /usr/src/php/ext && pecl download $PACKAGE_LIST && \
    for PACKAGE in $PACKAGE_LIST; do tar xzf $PACKAGE-* && \
    rm $PACKAGE*.tgz && mv $PACKAGE-* $PACKAGE; done

# Enable OPCache
RUN chmod +x /usr/local/bin/docker-php-ext-* && sync \
    && docker-php-ext-install $PACKAGE_LIST \
    && echo "zend_extension=opcache.so"> /usr/local/etc/php/conf.d/opcache.ini

# Clean the temporary files
RUN rm -rf /usr/src/php/ext/* /var/tmp/* \
            /tmp/* /var/lib/apt/lists/* /usr/src/php/ext/* \
            /etc/ssh/ssh_host_* /usr/share/man/?? \
            /usr/share/man/??*

WORKDIR /var/www/html
COPY php-fpm.conf /usr/local/etc/

EXPOSE 9000
CMD ["php-fpm"]
