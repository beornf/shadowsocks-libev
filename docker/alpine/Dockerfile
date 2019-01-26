#
# Dockerfile for shadowsocks-libev
#

FROM alpine:edge
LABEL maintainer="kev <noreply@datageek.info>, Sah <contact@leesah.name>"

ENV GOPATH /opt/go
ENV PATH $GOPATH/bin:$PATH
ENV SERVER_ADDR 0.0.0.0
ENV SERVER_ADDR_IPV6 ::0
ENV SERVER_PORT 8388
ENV PASSWORD=
ENV METHOD      aes-256-gcm
ENV TIMEOUT     300
ENV DNS_ADDRS    8.8.8.8,8.8.4.4
ENV ARGS=
ENV PLUGIN v2ray-plugin
ENV PLUGIN_OPTS server

COPY . /tmp/repo
RUN set -ex \
 # Build environment setup
 && apk add --no-cache --virtual .build-deps \
      autoconf \
      automake \
      build-base \
      c-ares-dev \
      git \
      go \
      libev-dev \
      libtool \
      libsodium-dev \
      linux-headers \
      mbedtls-dev \
      pcre-dev \
 # Build & install
 && cd /tmp/repo \
 && ./autogen.sh \
 && ./configure --prefix=/usr --disable-documentation \
 && make install \
 && cd .. \
 && git clone https://github.com/shadowsocks/simple-obfs \
 && cd simple-obfs \
 && git submodule update --init --recursive \
 && ./autogen.sh \
 && ./configure --prefix=/usr --disable-documentation \
 && make install \
 && go get -v github.com/shadowsocks/v2ray-plugin \
 && apk del .build-deps \
 # Runtime dependencies setup
 && apk add --no-cache \
      rng-tools \
      $(scanelf --needed --nobanner /usr/bin/ss-* \
      | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
      | sort -u) \
 && rm -rf /tmp/repo /opt/go/src

USER nobody

CMD exec ss-server \
      -s $SERVER_ADDR \
      # -s $SERVER_ADDR_IPV6 \
      -p $SERVER_PORT \
      -k ${PASSWORD:-$(hostname)} \
      -m $METHOD \
      -t $TIMEOUT \
      # --fast-open \
      --plugin $PLUGIN \
      --plugin-opts $PLUGIN_OPTS \
      -d $DNS_ADDRS \
      $ARGS
