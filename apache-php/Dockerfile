FROM almalinux:8.5

RUN dnf -y update --exclude=kernel* --exclude=centos*

# 言語を日本語に設定
RUN dnf -y install glibc-locale-source glibc-langpack-ja
RUN localedef -i ja_JP -f UTF-8 ja_JP.UTF-8
RUN echo 'LANG="ja_JP.UTF-8"' >  /etc/locale.conf
ENV LANG ja_JP.UTF-8

# 日付を日本語に設定
RUN echo 'ZONE="Asia/Tokyo"' > /etc/sysconfig/clock
RUN rm -f /etc/localtime
RUN ln -fs /usr/share/zoneinfo/Asia/Tokyo /etc/localtime

# 外部リポジトリ（EPEL, Remi）を追加
RUN dnf -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
RUN dnf -y install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
RUN dnf -y install dnf-utils

# apache その他 phpパッケージを導入
RUN dnf -y install httpd cronie-noanacron
RUN dnf -y module reset php
RUN dnf -y module install php:remi-7.4
RUN dnf -y install \
    php-cli \
    php-common \
    php-devel \
    php-fpm \
    php-gd \
    php-mbstring \
    php-mysqlnd \
    php-pdo \
    php-pear \
    php-pecl-apcu \
    php-pecl-zip \
    php-opcache \
    php-soap \
    php-xml \
    php-xmlrpc \
    zip \
    unzip \
    cmake \
    gtk2-devel \
    libjpeg-devel \
    libpng-devel && \
    rm -rf /var/cache/dnf/* && \
    dnf clean all && \
    systemctl enable httpd

# composerとnodeのインストール
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
COPY --from=node:lts /usr/local/bin /usr/local/bin
COPY --from=node:lts /usr/local/lib /usr/local/lib

# cronファイルコピー
COPY my.cron /var/spool/cron/root

# 各環境変数を設定
ENV USER oc-user
ENV HOME /home/${USER}
ENV SHELL /bin/bash
ENV UID 1000

# 一般ユーザアカウントを追加
RUN useradd -m -u ${UID} ${USER}
# 一般ユーザにグループ追加
RUN usermod -aG apache ${USER}

# OpenCVのファイルを追加
ADD opencv-4.5.3.tar.gz /home/oc-user/opencv_build/
ADD opencv_contrib-4.5.3.tar.gz /home/oc-user/opencv_build/

# OpenCVインストール
RUN cd /home/oc-user/opencv_build/opencv-4.5.3/ && \
    mkdir build && \
    cd build/ && \
    cmake -DOPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-4.5.3/modules .. && \
    make -j8 && \
    make install && \
    ldconfig

WORKDIR /var/www/sample

EXPOSE 80
