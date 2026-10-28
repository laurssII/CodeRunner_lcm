FROM moodlehq/moodle-php-apache:8.1

# Actualizar los repositorios
RUN apt-get update

# Instalar las extensiones necesarias de PHP 8.1
RUN apt-get install -y --no-install-recommends \
    git \
    unzip \
    curl \
    libonig-dev \
    libxml2-dev \
    libcurl4-openssl-dev \
    && rm -rf /var/lib/apt/lists/*

#Instalar extensiones php 8.1:
RUN docker-php-ext-install mbstring xml curl

# Clonar el repositorio de Moodle
RUN git clone -b MOODLE_405_STABLE https://github.com/moodle/moodle.git /var/www/html/moodle

RUN rm -f /var/www/html/moodle/index.html
RUN find /var/www/html/moodle -type f -exec chmod 644 {} \;
RUN find /var/www/html/moodle -type d -exec chmod 755 {} \;
RUN curl -sS https://getcomposer.org/installer | php && \
              mv composer.phar /usr/local/bin/composer

