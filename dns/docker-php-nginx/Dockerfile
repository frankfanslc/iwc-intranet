#-------------------------------------
# v1.0.0 build for IWC Intranet
#
#ARM64 DOCKER ALPINE NGINX PHP-FPM RTMP FFMPEG XDEBUG
# **********************************
# ADAN : PRFX - Benchmarking build
# **********************************
# A) Custom builds implementing Debian Stretch and Alpine aarch64/arm64 CPU targeting
# D) A completed Docker image that runs web.conf configurations and Laravel on entry of the container, to initiate Nginx, run custom entrypoint scripts, and debug remotely
# A) Debian Stretch builder output files are copied to a final php:fpm-alpine container 
# N) Nginx custom built with all supported modules, including OpenSSL, nginx-rtmp, and gperftools for performance monitoring
# P) PHP-FPM for other Nginx instances to connect to for computing tasks, with Composer, Laravel and XDEBUG support
# R) The FOSS RTMP Nginx module built into Nginx
# F) FFMPEG custom build for multimedia streaming tasks
# X) XDEBUG, and eXpansion through additional entrypoint scripts and Nginx configurations    
#
#
# !PRIMARY HTTP/SSL SUPERVISOR FOR ALL IWC INTRANET WEB APPS!
# 
#
# THERE BE DRAGONS
#  -- This is an experimental Docker staged build -- no custom community universes/repos are used except for the nginx rtmp module and Alpine edge 
#  -- Master is regularly patched to assist in updating containers already running in the Cloud
#  -- If you find a release of this file that works for you, it's probably best to stay at that version for a while unless you like regression
#  -- This build is contradictory by implementation:
#       Alpine is a small OS package
#       but is the largest Nginx package possible, 
#       with every officially supported module.
#  -- There are much, much smaller Nginx packages out there
#  -- There are also more performant Nginx packages everywhere
#  -- There are no other easy-to-find Docker Nginx packages that support Alpine on aarch64/arm64 devices, with all performance and dev tools enabled. Community support is limited.
#  -- Long live the benchmark; fight them dragons with pull requests, please
#
#
# COMPONENTS/SERVICES
#  - PHP 7.3.4 FPM as default; set with ENV
#  - Nginx 1.15.11; set with ENV
#  - RTMP module 1.2.1 (probably will be for a very long time, but you can set with ENV)
#  - FFMPEG 4.1.3 with DASH/MP4/5 support; set with ENV
#  - XDebug
#  - Based on arm64v8 group's aarch64 Docker repo for easy migration from existing images
#  - OpenSSL is baked into nginx and ffmpeg for tighter reverse proxy integrations and encrypted multimedia streaming
#
#----------------------------

##
# From Debian Stretch as a factory
##
FROM debian:stretch-slim AS stage0

LABEL maintainer="Jen McQuade <info@indiewebconsulting.com>"

#
#--------------------------------------------------------------------------
# Install nginx using Debian as a build environment. Alpine has musl bugs when compiling with google performance tools.
#--------------------------------------------------------------------------
#
WORKDIR /build
ENV NGINX_VERSION 1.15.11
ENV NGINX_RTMP_VERSION 1.2.1
ENV FFMPEG_VERSION 4.1.3

USER root

# Build dependencies.
RUN \
  apt-get update && apt-get install --fix-missing -y \
  wget \
  build-essential \
  autoconf \
  openssl \
  curl \
  libc6 \
  gcc \
  git \
  unzip \
  libc-dev \
  autoconf \
  automake \
  make \
  musl-dev \
  openssl \
  pkgconf \
  libtool \ 
  zlib1g-dev \
  libpcre3 \
  libpcre3-dev \
  libssl-dev \
  libxslt1-dev libxml2-dev \
  libgd2-xpm-dev \
  libgeoip-dev \
  libperl-dev \
  libgoogle-perftools-dev \
  libunwind-dev 

# Get nginx, nginx-rtmp module, openssl, gperfools, and brotli source code.
RUN \
  wget http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz \
  && tar zxf nginx-${NGINX_VERSION}.tar.gz \
  && rm nginx-${NGINX_VERSION}.tar.gz \
  && git clone https://github.com/openssl/openssl.git \
  && cd openssl \
  # This commit is 'special', because it works
  && git checkout 5dd94f1847c744929a3bd24819f1c99644bb18c7 \
  && cd /build \
  && git clone https://github.com/arut/nginx-rtmp-module.git \
  && git clone https://github.com/gperftools/gperftools.git \ 
  && git clone --recursive https://github.com/google/ngx_brotli.git \
  && cd ngx_brotli \
  && git submodule update --init --recursive 

##
# Now we have a Debian Stretch build environment with the right libraries. So, let's build!
##

FROM stage0 AS gperftools 

# Compile gperftools
RUN \
  cd /build/gperftools \
  && sh ./autogen.sh \
  && ./configure \
  && make \
  && make install 

# Compile nginx with nginx-rtmp module, openssl, gperftools and brotli libraries.
# DEV NOTE: You can set the $CFLAGS in your environment or docker-compose.yml to modify GCC build configurations, if you get warnings that break the Docker build.
# By far, this is the most breakable stage of the build
FROM gperftools AS build-nginx
RUN \
  cd /build/nginx-${NGINX_VERSION} \
  && ./configure \
  --prefix=/usr/local/nginx \
  --sbin-path=/usr/sbin/nginx \
  --conf-path=/etc/nginx/nginx.conf \
  --error-log-path=/dev/stdout \
  --http-log-path=/dev/stdout \
  --pid-path=/var/run/nginx.pid \
  --lock-path=/var/run/nginx.lock \
  --http-client-body-temp-path=/var/cache/nginx/client_temp \
  --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
  --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
  --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
  --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
  --user=www \
  --group=www \
  --add-module=/build/nginx-rtmp-module \
  --add-module=/build/ngx_brotli \
  --with-threads \
  --with-file-aio \
  --with-http_ssl_module \ 
  --with-openssl=/build/openssl \
  --with-http_v2_module \
  --with-http_realip_module \
  --with-http_addition_module \
  --with-http_sub_module \
  --with-http_dav_module \
  --with-http_flv_module \
  --with-http_mp4_module \
  --with-http_gunzip_module \
  --with-http_gzip_static_module \
  --with-http_auth_request_module \
  --with-http_random_index_module \
  --with-http_secure_link_module \
  --with-http_degradation_module \
  --with-http_slice_module \
  --with-http_stub_status_module \
  --with-ipv6 \
  --with-mail \
  --with-mail_ssl_module \
  --with-stream \
  --with-stream_ssl_module \
  --with-stream_realip_module \
  --with-stream_ssl_preread_module \
  --with-cpp_test_module \
  --with-compat \
  --with-cc-opt=${CFLAGS} \
  --with-google_perftools_module \
  --error-log-path=/etc/nginx/logs/error.log \
  --http-log-path=/etc/nginx/logs/access.log \
  --with-debug \
   && make \
   && make install \ 
   && rm -rf /var/cache/* /build/*

ADD ./conf /etc/nginx/conf.d

###############################
# Build the FFmpeg-build image.
# This doesn't require any stage0 components, so we're OK to build using Alpine instead of Debian
##############################
FROM alpine as build-ffmpeg
ARG FFMPEG_VERSION
ARG PREFIX=/usr/local
ARG MAKEFLAGS="-j4"
ENV FFMPEG_VERSION 4.1.3

# FFmpeg build dependencies.
RUN apk add --update \
  build-base \
  freetype-dev \
  lame-dev \
  libogg-dev \
  libass \
  libass-dev \
  libressl-dev \
  libvpx-dev \
  libvorbis-dev \
  libwebp-dev \
  libtheora-dev \
  opus-dev \
  pkgconf \
  pkgconfig \
  rtmpdump-dev \
  wget \
  x264-dev \
  x265-dev \
  yasm

##
# Alpine Edge repo for aac support
##
RUN echo http://dl-cdn.alpinelinux.org/alpine/edge/testing >> /etc/apk/repositories
RUN apk --update add fdk-aac-dev

# Get FFmpeg source.
RUN mkdir /build && cd /build && \
  wget http://ffmpeg.org/releases/ffmpeg-${FFMPEG_VERSION}.tar.gz && \
  tar zxf ffmpeg-${FFMPEG_VERSION}.tar.gz && rm ffmpeg-${FFMPEG_VERSION}.tar.gz

# Compile ffmpeg.
RUN cd /build/ffmpeg-${FFMPEG_VERSION} && \
  ./configure \
  --prefix=${PREFIX} \
  --enable-version3 \
  --enable-gpl \
  --enable-nonfree \
  --enable-small \ 
  --enable-libmp3lame \
  --enable-libx264 \
  --enable-libx265 \
  --enable-libvpx \
  --enable-libtheora \
  --enable-libvorbis \
  --enable-libopus \
  --enable-libfdk-aac \
  --enable-libass \
  --enable-libwebp \
  --enable-librtmp \
  --enable-postproc \
  --enable-avresample \
  --enable-libfreetype \
  --enable-openssl \
  --disable-debug \
  --disable-doc \
  --disable-ffplay \
  --extra-libs="-lpthread -lm" \
  && make USE_SYSTEM_LIBM=1 \
  && make install 

# Cleanup.
RUN rm -rf /var/cache/* /build/*

##########################
# Build the releasable image.
FROM php:7.3.4-fpm-alpine as stage1

LABEL MAINTAINER Jen McQuade <info@indiewebconsulting.com>

ENV CERT_DIR /etc/letsencrypt/live

RUN apk update \
  && apk add \
  ca-certificates \
  openssl \
  curl \
  pcre \ 
  libxml2 \
  lame \
  libogg \
  libass \
  libvpx \
  libvorbis \
  libwebp \
  libtheora \
  opus \
  rtmpdump \
  x264-dev \
  x265-dev

## Composer for PHP dev support
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer

COPY --from=build-nginx /etc/nginx /etc/nginx
COPY --from=build-ffmpeg /usr/local /usr/local
COPY --from=build-ffmpeg /usr/lib/libfdk-aac.so.2 /usr/lib/libfdk-aac.so.2

RUN \
  apk --update add \
  certbot \ 
  # PHP MODULE INSTALLER PREREQS
  libxml2-dev \
  freetype-dev \
  libjpeg-turbo-dev \
  libpng-dev \
  bzip2-dev \ 
  net-snmp-dev \
  php7-tidy \   
  tidyhtml-dev \ 
  libxslt-dev \
  libzip-dev

# ADDITIONAL PHP MODULES
RUN \
  docker-php-ext-configure gd --with-freetype-dir=/usr/include --with-jpg-dir=/usr/include && \ 
  docker-php-ext-install -j$(nproc) mysqli snmp soap sockets tidy tokenizer xml xmlrpc xsl zip gd
  
# Add NGINX config and static files.
ADD conf/nginx.conf /etc/nginx/nginx.conf
RUN mkdir -p /opt/data ; mkdir -p /var/www/html
ADD app /var/www/html

# Ensure we keep some folders open for sharing
VOLUME ${CERT_DIR}
VOLUME /etc/nginx/conf.d
VOLUME /opt/data
VOLUME /var/www/html

COPY certs ${CERT_DIR}

#
#--------------------------------------------------------------------------
# Applications and Entrypoints
#--------------------------------------------------------------------------
#

ADD ./config/bin/schedule /etc/crontabs/root

ADD ./config/supervisord/* /etc/supervisord/

ADD ./config/entrypoints/* /var/entrypoints/

ENV TZ='America/Los Angeles'

ENV PATH="$PATH:/var/www/html"

WORKDIR /var/www/html

RUN mkdir -p /var/log/cron && touch /var/log/cron/cron.log

ENV SCHEDULE_LOG_PATH /var/log/cron/cron.log

#
#--------------------------------------------------------------------------
# Init
#--------------------------------------------------------------------------
#

ENTRYPOINT ["/var/entrypoints/laravel"]

CMD ["/usr/bin/supervisord","-n","-c","/etc/supervisord/web.conf"]

RUN mkdir /var/cache/nginx \
  && chown root:root /var/cache/nginx

# Add prerequisites for xdebug
RUN apk add --no-cache $PHPIZE_DEPS

# Install xdebug, redis using pecl -- Always use version numbers for these
RUN pecl install xdebug-2.7.0 \
  && pecl install redis-4.0.1 \
  && docker-php-ext-enable redis xdebug

# Configure xdebug for remote debugging with PhpStorm
RUN echo "zend_extension=$(find /usr/local/lib/php/extensions/ -name xdebug.so)" > /usr/local/etc/php/conf.d/xdebug.ini \
 && echo "xdebug.remote_enable=on" >> /usr/local/etc/php/conf.d/xdebug.ini \
 && echo "xdebug.remote_autostart=on" >> /usr/local/etc/php/conf.d/xdebug.ini \
 && echo "xdebug.remote_host=docker.for.mac.localhost" >> /usr/local/etc/php/conf.d/xdebug.ini \
 && echo "xdebug.remote_port=9000" >> /usr/local/etc/php/conf.d/xdebug.ini \
 && echo "xdebug.remote_log=/var/www/storage/logs/xdebug.log" >> /usr/local/etc/php/conf.d/xdebug.ini

FROM stage1

## Really just noting here the ports the IWC Intranet uses for rproxy services.  We usually take care of these with port mapping in docker-compose.yml files instead of directly using EXPOSE
EXPOSE 80 443 1935 5900-5911 8080 3389 9000

