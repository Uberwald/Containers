FROM docker.io/nextcloud:stable-apache

RUN apt-get update \
    && apt-get install -y \
        curl \
        vim \
    && rm -rf /var/lib/apt/lists/*

# COPY <локальный_файл_или_директория> <путь_в_контейнере>

# RUN sed -i 's/Listen 80/Listen 8080/' /etc/apache2/ports.conf

CMD ["apache2-foreground"]
