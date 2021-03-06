FROM php:5.6-apache

# install the PHP extensions we need
RUN set -ex; \
	\
	apt-get update; \
	apt-get install -y \
		libjpeg-dev \
		libpng-dev \
		libmemcached-dev \
		php5-memcached \
	; \
	rm -rf /var/lib/apt/lists/*; \
	\
	docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr; \
	docker-php-ext-install gd mysqli opcache 
# TODO consider removing the *-dev deps and only keeping the necessary lib* packages

RUN pecl install memcached-2.2.0 memcache
RUN echo extension=memcached.so >> /usr/local/etc/php/conf.d/memcached.ini
RUN echo extension=memcache.so >> /usr/local/etc/php/conf.d/memcached.ini

ENV MEMCACHED_HOST memcached

RUN { \
		echo 'session.save_handler = "memcached"' ; \
		echo "session.save_path = ${MEMCACHED_HOST}:11211" ; \
	} >> /usr/local/etc/php/conf.d/memcached.ini

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=2'; \
		echo 'opcache.fast_shutdown=1'; \
		echo 'opcache.enable_cli=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini

RUN a2enmod rewrite expires

VOLUME /var/www/html

ENV WORDPRESS_VERSION 4.8.3
#ENV WORDPRESS_VERSION 4.9.1
#ENV WORDPRESS_SHA1 892d2c23b9d458ec3d44de59b753adb41012e903

RUN set -ex; \
	curl -o wordpress.tar.gz -fSL "https://wordpress.org/wordpress-${WORDPRESS_VERSION}.tar.gz"; \
# upstream tarballs include ./wordpress/ so this gives us /usr/src/wordpress
	tar -xzf wordpress.tar.gz -C /usr/src/; \
	rm wordpress.tar.gz; \
	chown -R www-data:www-data /usr/src/wordpress

COPY docker-entrypoint.sh /usr/local/bin/

RUN chmod +x /usr/local/bin/docker-entrypoint.sh

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
CMD ["apache2-foreground"]
