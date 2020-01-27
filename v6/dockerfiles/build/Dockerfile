FROM ubuntu:18.04

ENV TZ=America/Los_Angeles
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN apt-get update -y -qq; apt-get install --no-install-recommends -y -qq software-properties-common; \
    LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php -y; apt-get update -y -qq; \
    apt-get install --no-install-recommends -y -qq nginx sudo supervisor php7.3-fpm php7.3-common php7.3-mbstring php7.3-xmlrpc php7.3-soap \
    php7.3-gd php7.3-xml php7.3-intl php7.3-mysql php7.3-cli php7.3-zip php7.3-bz2 php7.3-curl php7.3-imagick php7.3-apcu \
    php7.3-redis libreoffice libreoffice-math xfonts-75dpi poppler-utils inkscape libxrender1 libfontconfig1 \
    ghostscript python-setuptools python-pip libimage-exiftool-perl cron golang-go npm unzip git procps wget; \
    easy_install uno; \
    npm install -g sqip; \
    rm -rf /etc/nginx/sites-available/default; rm -rf /etc/nginx/sites-enabled/default;  mkdir /run/php; \
    mkdir -p /opt/letsencrypt; \
    wget https://github.com/pimcore/pimcore/archive/v6.4.2.zip -O /usr/src/pimcore-install.zip; \
    wget https://johnvansickle.com/ffmpeg/builds/ffmpeg-git-64bit-static.tar.xz; \
    tar -Jxf ffmpeg*.tar.xz; \
    rm ffmpeg*.tar.xz; \
    mv ffmpeg-* /usr/local/ffmpeg; \
    ln -s /usr/local/ffmpeg/ffmpeg /usr/local/bin/; \
    ln -s /usr/local/ffmpeg/ffprobe /usr/local/bin/; \
    ln -s /usr/local/ffmpeg/qt-faststart /usr/local/bin/; \
    ln -s /usr/local/ffmpeg/qt-faststart /usr/local/bin/qtfaststart; \
    wget https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.4/wkhtmltox-0.12.4_linux-generic-amd64.tar.xz; \
    tar -Jxf wkhtmltox*.tar.xz; \
    rm -rf wkhtmltox*.tar.xz ; \
    mv wkhtmltox /usr/local/; \
    ln -s  /usr/local/wkhtmltox/bin/wkhtmltoimage  /usr/local/bin/wkhtmltoimage; \
    wget https://files.pythonhosted.org/packages/dd/79/f8387c4e82275a7b540e0b948d261a636eb5aedd1d23be8ca05fbf605726/html2text-2018.1.9.tar.gz; \
    tar -xzf  html2text*.tar.gz; \
    rm -rf html2text*.tar.gz; \
    cd html2text* ; python setup.py install; \
    rm -rf /html2text*; \
    wget https://github.com/imagemin/zopflipng-bin/raw/master/vendor/linux/zopflipng -O /usr/local/bin/zopflipng; \
    chmod 0755 /usr/local/bin/zopflipng; \
    wget https://github.com/imagemin/pngcrush-bin/raw/master/vendor/linux/pngcrush -O /usr/local/bin/pngcrush; \
    chmod 0755 /usr/local/bin/pngcrush; \
    wget https://github.com/imagemin/jpegoptim-bin/raw/master/vendor/linux/jpegoptim -O /usr/local/bin/jpegoptim; \
    chmod 0755 /usr/local/bin/jpegoptim; \
    wget https://github.com/imagemin/pngout-bin/raw/master/vendor/linux/x64/pngout -O /usr/local/bin/pngout; \
    chmod 0755 /usr/local/bin/pngout; \
    wget https://github.com/imagemin/advpng-bin/raw/master/vendor/linux/advpng -O /usr/local/bin/advpng; \
    chmod 0755 /usr/local/bin/advpng; \
    wget https://github.com/imagemin/mozjpeg-bin/raw/master/vendor/linux/cjpeg -O /usr/local/bin/cjpeg; \
    chmod 0755 /usr/local/bin/cjpeg;

RUN echo "opcache.enable=1" >> /etc/php/7.3/fpm/php.ini && \
    echo "opcache.enable=1" >> /etc/php/7.3/cli/php.ini && \
    phpenmod opcache && \
    php -i | grep opcache

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

RUN groupadd -r -g 2000 pimcore; useradd -r -u 2000 -g 2000 -m -c "app account" -d /home/pimcore -s /bin/bash pimcore
RUN chown -R pimcore. /usr/src /var/www/html

COPY ./configs/php/fpm/php.ini /etc/php/7.3/fpm/php.ini
COPY ./configs/php/www.conf /etc/php/7.3/fpm/pool.d/www.conf
COPY ./configs/supervisord/pimcore.conf /etc/supervisor/conf.d/pimcore.conf
COPY ./scripts/letsencrypt-auto /opt/letsencrypt/letsencrypt-auto
RUN chmod +x /opt/letsencrypt/letsencrypt-auto; /opt/letsencrypt/letsencrypt-auto  --os-packages-only --install-only --non-interactive

ENV GOPATH="/usr/local"
ENV PATH=$PATH:$GOPATH/bin

RUN go get -u github.com/fogleman/primitive; apt-get purge -qq -y npm software-properties-common; \
    apt-get clean; apt-get autoclean; \
    apt-get autoremove -y -qq

USER pimcore
RUN composer global require hirak/prestissimo
WORKDIR /var/www/html

# Too long to wait
RUN COMPOSER_MEMORY_LIMIT=-1 composer create-project pimcore/demo pimcore --no-dev
RUN mv /var/www/html/pimcore/*  /var/www/html
RUN chmod -R gou+w /var/www/html/web/var

USER root
RUN touch /var/run/nginx.pid && \
  mkdir -p /var/cache/nginx && \
  chown -R pimcore:pimcore /var/run/nginx.pid && \
  chown -R pimcore:pimcore /var/cache/nginx && \
  chown -R pimcore:pimcore /var/log

USER pimcore