FROM php:7.0-apache
RUN apt-get update && apt-get -y install libcurl4-openssl-dev && apt-get -y install libxml2-dev
RUN curl -O 'http://pecl.php.net/get/solr-2.4.0.tgz' && tar -xvf solr-2.4.0.tgz && cd solr-2.4.0 && phpize && ./configure && make && make install && docker-php-ext-enable solr
COPY html/ /var/www/html/
