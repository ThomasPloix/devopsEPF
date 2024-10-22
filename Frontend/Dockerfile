FROM httpd:2.4

COPY ./public-html/ /usr/local/apache2/htdocs/

COPY proxy.conf /tmp/proxy.conf

RUN cat /tmp/proxy.conf >> /usr/local/apache2/conf/httpd.conf