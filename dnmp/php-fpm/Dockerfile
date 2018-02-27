FROM laradock/php-fpm:2.0-71

MAINTAINER Mahmoud Zalt <mahmoud@zalt.me>



#####################################
# Check PHP version:
#####################################
RUN php -v | head -n 1 | grep -q "PHP 7.1."


#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

ADD ./laravel.ini /usr/local/etc/php/conf.d
ADD ./xlaravel.pool.conf /usr/local/etc/php-fpm.d/

#RUN rm -r /var/lib/apt/lists/*

RUN usermod -u 1000 www-data

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000