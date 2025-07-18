# syntax=docker/dockerfile-upstream:master
FROM alpine:3.16 as boringssl-builder

ARG ALPINE_URL=dl-cdn.alpinelinux.org

ARG OPENSSL_BRANCH=OpenSSL_1_1_1a

ARG OPENSSL_URL=https://github.com/google/boringssl

RUN set -x \
     && sed -i "s/dl-cdn.alpinelinux.org/${ALPINE_URL}/g" /etc/apk/repositories \
     && apk add --no-cache --virtual .build-deps \
        git \
        cmake \
        samurai \
        libstdc++ \
        build-base \
        go \
        perl-dev \
        linux-headers \
        libunwind-dev \
     # && curl -fSL https://github.com/google/boringssl/archive/master.tar.gz -o boringssl.tar.gz \
     && git clone --depth=1 -b master ${OPENSSL_URL} /usr/src/boringssl \
     && cd /usr/src/boringssl \
     && mkdir build \
     && cd build \
     && cmake -GNinja .. \
     && ninja \
     && ls -la \
     && ls -la ../include \
     && apk del --no-network .build-deps

FROM alpine:3.16

LABEL maintainer="khs1994.com nginx With HTTP3"

ARG NGINX_VERSION=1.27.0

ARG ALPINE_URL=dl-cdn.alpinelinux.org

RUN --mount=type=bind,from=boringssl-builder,source=/usr/src/boringssl,target=/usr/src/boringssl \
  set -x \
     && sed -i "s/dl-cdn.alpinelinux.org/${ALPINE_URL}/g" /etc/apk/repositories \
     && GPG_KEYS=B0F4253373F8F6F510D42178520A9993A1C052F8 \
     && CONFIG="\
    --prefix=/etc/nginx \
    --sbin-path=/usr/sbin/nginx \
    --modules-path=/usr/lib/nginx/modules \
    --conf-path=/etc/nginx/nginx.conf \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --pid-path=/var/run/nginx.pid \
    --lock-path=/var/run/nginx.lock \
    --http-client-body-temp-path=/var/cache/nginx/client_temp \
    --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
    --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
    --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
    --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
    --user=nginx \
    --group=nginx \
    --with-http_ssl_module \
    --with-http_realip_module \
    --with-http_addition_module \
    --with-http_sub_module \
    --with-http_dav_module \
    --with-http_flv_module \
    --with-http_mp4_module \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_random_index_module \
    --with-http_secure_link_module \
    --with-http_stub_status_module \
    --with-http_auth_request_module \
    --with-http_xslt_module=dynamic \
    --with-http_image_filter_module=dynamic \
    --with-http_geoip_module=dynamic \
    --with-threads \
    --with-stream \
    --with-stream_ssl_module \
    --with-stream_ssl_preread_module \
    --with-stream_realip_module \
    --with-stream_geoip_module=dynamic \
    --with-http_slice_module \
    --with-mail \
    --with-mail_ssl_module \
    --with-compat \
    --with-file-aio \
    --with-http_v2_module \
    --with-http_v3_module \
    --with-http_quic_module \
    --with-stream_quic_module \
  " \
  && addgroup -S nginx \
  && adduser -D -S -h /var/cache/nginx -s /sbin/nologin -G nginx nginx \
  && apk add --no-cache --virtual .build-deps \
    gcc \
    libc-dev \
    make \
    # openssl-dev \
    pcre-dev \
    zlib-dev \
    linux-headers \
    curl \
    gnupg1 \
    libxslt-dev \
    gd-dev \
    geoip-dev \
    perl-dev \
  && curl -fSL https://hg.nginx.org/nginx-quic/archive/quic.tar.gz -o nginx.tar.gz \
  # && curl -fSL https://nginx.org/download/nginx-$NGINX_VERSION.tar.gz.asc -o nginx.tar.gz.asc \
  # && export GNUPGHOME="$(mktemp -d)" \
  # && found=''; \
  # for server in \
  #   ha.pool.sks-keyservers.net \
  #   hkp://keyserver.ubuntu.com:80 \
  #   hkp://p80.pool.sks-keyservers.net:80 \
  #   pgp.mit.edu \
  # ; do \
  #   echo "Fetching GPG key $GPG_KEYS from $server"; \
  #   gpg --keyserver "$server" --keyserver-options timeout=10 --recv-keys "$GPG_KEYS" && found=yes && break; \
  # done; \
  # test -z "$found" && echo >&2 "error: failed to fetch GPG key $GPG_KEYS" && exit 1; \
  # gpg --batch --verify nginx.tar.gz.asc nginx.tar.gz \
  # && rm -rf "$GNUPGHOME" nginx.tar.gz.asc \
  && mkdir -p /usr/src \
  && tar -zxC /usr/src -f nginx.tar.gz \
  # && tar -zxC /usr/src -f boringssl.tar.gz \
  && rm -rf nginx.tar.gz boringssl.tar.gz \
  && cd /usr/src/nginx-quic-quic \
  && ./auto/configure $CONFIG --with-debug \
       --with-cc-opt="-I../boringssl/include" \
       --with-ld-opt="-L../boringssl/build/ssl  -L../boringssl/build/crypto" \
  && make -j$(getconf _NPROCESSORS_ONLN) \
  && mv objs/nginx objs/nginx-debug \
  && mv objs/ngx_http_xslt_filter_module.so objs/ngx_http_xslt_filter_module-debug.so \
  && mv objs/ngx_http_image_filter_module.so objs/ngx_http_image_filter_module-debug.so \
  && mv objs/ngx_http_geoip_module.so objs/ngx_http_geoip_module-debug.so \
  && mv objs/ngx_stream_geoip_module.so objs/ngx_stream_geoip_module-debug.so \
  && ./auto/configure $CONFIG \
       --with-cc-opt="-I../boringssl/include" \
       --with-ld-opt="-L../boringssl/build/ssl  -L../boringssl/build/crypto" \
  && make -j$(getconf _NPROCESSORS_ONLN) \
  && make install \
  && rm -rf /etc/nginx/html/index.html \
  && mkdir /etc/nginx/conf.d/ \
  # && mkdir -p /usr/share/nginx/html/ \
  && install -m755 objs/nginx-debug /usr/sbin/nginx-debug \
  && install -m755 objs/ngx_http_xslt_filter_module-debug.so /usr/lib/nginx/modules/ngx_http_xslt_filter_module-debug.so \
  && install -m755 objs/ngx_http_image_filter_module-debug.so /usr/lib/nginx/modules/ngx_http_image_filter_module-debug.so \
  && install -m755 objs/ngx_http_geoip_module-debug.so /usr/lib/nginx/modules/ngx_http_geoip_module-debug.so \
  && install -m755 objs/ngx_stream_geoip_module-debug.so /usr/lib/nginx/modules/ngx_stream_geoip_module-debug.so \
  && ln -s /usr/lib/nginx/modules /etc/nginx/modules \
  && strip /usr/sbin/nginx* \
  && strip /usr/lib/nginx/modules/*.so \
  && rm -rf /usr/src/nginx-quic-quic \
# Bring in gettext so we can get `envsubst`, then throw
# the rest away. To do this, we need to install `gettext`
# then move `envsubst` out of the way so `gettext` can
# be deleted completely, then move `envsubst` back.
  && apk add --no-cache --virtual .gettext gettext \
  && mv /usr/bin/envsubst /tmp/ \
  \
  && runDeps="$( \
    scanelf --needed --nobanner --format '%n#p' /usr/sbin/nginx /usr/lib/nginx/modules/*.so /tmp/envsubst \
      | tr ',' '\n' \
      | sort -u \
      | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
  )" \
  && apk add --no-cache --virtual .nginx-rundeps $runDeps \
  && apk del --no-network .build-deps \
  && apk del --no-network .gettext \
  && mv /tmp/envsubst /usr/local/bin/ \
  \
# Bring in tzdata so users could set the timezones through the environment
# variables
  && apk add --no-cache tzdata \
# Bring in curl and ca-certificates to make registering on DNS SD easier
  && apk add --no-cache curl ca-certificates \
  \
# forward request and error logs to docker log collector
  && ln -sf /dev/stdout /var/log/nginx/access.log \
  && ln -sf /dev/stderr /var/log/nginx/error.log

COPY nginx.conf /etc/nginx/nginx.conf
COPY nginx.vh.default.conf /etc/nginx/conf.d/default.conf
COPY index.html /etc/nginx/html/

EXPOSE 80/tcp 443/tcp 443/udp

STOPSIGNAL SIGTERM

CMD ["nginx", "-g", "daemon off;"]
