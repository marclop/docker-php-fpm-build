FROM php:5.5-fpm

MAINTAINER Marc Lopez <marc5.12@outlook.com>

### Package and dependencies stuff ###

ENV PACKAGE_LIST redis geoip igbinary xdebug solr zendopcache

# Install module dependencies
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y -q \
						libmcrypt4 libmhash2 libmcrypt-dev \
						libpng12-dev \
						libicu-dev \
						libbz2-dev \
						libgmp-dev \
						libssl-dev \
						libxslt1-dev \
						libcurl4-gnutls-dev \
						git \
						## g++ is a requirement for intl and need to clean up that later, it's a fat motherfucker (look into apt-get purge)
						g++ \
						geoip-database* libgeoip-dev \
						npm

# Symbolic link to GMP headers and nodeJS
RUN ln -s /usr/include/x86_64-linux-gnu/gmp.h /usr/include/gmp.h && \
	ln -s /usr/bin/nodejs /usr/bin/node

# Download source for external PHP extensions
RUN cd /usr/src/php/ext && pecl download $PACKAGE_LIST && \
	for PACKAGE in $PACKAGE_LIST; do tar xzf $PACKAGE-* && \
	rm $PACKAGE*.tgz && mv $PACKAGE-* $PACKAGE; done

# Install modules
RUN docker-php-ext-install \
							bcmath \
							bz2\
							calendar\
							curl \
							exif \
							fileinfo \
							gettext \
							geoip \
							gd \
							gmp \
							intl \
							igbinary \
							mbstring \
							mcrypt \
							mysql \
							mysqli \
							pcntl \
							pdo_mysql \
							shmop \
							sockets \
							solr \
							wddx \
							xsl \
							xdebug \
							zendopcache \
							zip

# Configure additional flags for redis extension, documented on issue: https://github.com/phpredis/phpredis/issues/335
RUN docker-php-ext-configure redis --enable-redis-igbinary && \
	docker-php-ext-install redis

# Clean the temporary files
RUN rm -rf /usr/src/php/ext/* /var/tmp/* \
			/tmp/* /var/lib/apt/lists/* /usr/src/php/ext/* \
			/usr/local/etc/php/conf.d/ext-curl.ini \
			/usr/local/etc/php/conf.d/ext-fileinfo.ini

