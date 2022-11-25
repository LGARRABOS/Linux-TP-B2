FROM debian

RUN apt update -y

RUN apt install -y apache2

RUN mkdir /etc/apache2/logs/

copy /apache2.conf /etc/apache2/apache2.conf

RUN rm /var/www/html/index.html

RUN echo Coucou > /var/www/html/index.html

EXPOSE 80
CMD ["apache2ctl", "-D", "FOREGROUND"]
