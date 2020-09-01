



```
 /usr/local/php/bin/phpize
 cat /usr/local/php/bin/php-config 
./configure --with-php-config=/usr/local/php/bin/php-config --with-curl=DIR
 make && make install
 echo 'extension=curl.so' >> /usr/local/php/php.ini
 
 /etc/init.d/php-fpm restart 或者 /usr/local/apache2/bin/apachectl restart


```



