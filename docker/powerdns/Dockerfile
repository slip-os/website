FROM alpine:3.15 AS build

ARG PDNS_VERSION=4.5.2

ENV PGSQL_HOST= \
    PGSQL_PORT=5432 \
    PGSQL_DATABASE= \
    PGSQL_USERNAME= \
    PGSQL_PASSWORD= \
    PDNS_CONF=/etc/pdns.conf \
    PDNS_CONF_DIR=/etc/pdns.d \
    PDNS_SQL_DIR=/var/lib/pdns \
    PDNS_MASTER=no \
    PDNS_SLAVE=no \
    PDNS_MASTER_ADDR= \
    PDNS_MASTER_NAME= \
    PDNS_WEBSERVER=no \
    PDNS_WEBSERVER_HOST=0.0.0.0 \
    PDNS_WEBSERVER_PORT=8081 \
    PDNS_WEBSERVER_ACL= \
    PDNS_API_KEY= \
    PDNS_AXFR_DISABLE=yes \
    PDNS_AXFR_ACL= \
    PDNS_LOG_LEVEL=6

ADD ./docker/powerdns/entrypoint.sh /entrypoint.sh
ADD ./docker/powerdns/pdns.conf.tmpl /etc/
ADD ./docker/powerdns/pgsql.conf.tmpl /etc/pdns.d/
ADD ./docker/powerdns/sqlite.conf.tmpl /etc/pdns.d/
RUN mkdir -p ${PDNS_CONF_DIR}
RUN mkdir -p ${PDNS_SQL_DIR}

RUN wget https://raw.githubusercontent.com/eficode/wait-for/master/wait-for
RUN chmod +x /wait-for
RUN chmod +x /entrypoint.sh
RUN wget https://raw.githubusercontent.com/PowerDNS/pdns/master/modules/gsqlite3backend/schema.sqlite3.sql -O ${PDNS_SQL_DIR}/schema.sqlite3.sql

RUN apk add --no-cache --repository=http://dl-cdn.alpinelinux.org/alpine/edge/testing/ envsubst && \
    apk add --no-cache build-base lua lua-dev libcrypto1.1 libssl1.1 \
                       openssl-dev boost boost-dev postgresql-client libpq-dev \
                       libcurl curl-dev sqlite sqlite-dev coreutils && \
    mkdir /build && cd /build && \
    wget https://downloads.powerdns.com/releases/pdns-${PDNS_VERSION}.tar.bz2 && \
    tar xjf pdns-${PDNS_VERSION}.tar.bz2 && \
    cd pdns-${PDNS_VERSION} && \
    ./configure --prefix=/ --with-modules="bind gpgsql gsqlite3 random" && \
    make -j 8 && \
    make install && \
    apk del build-base lua-dev openssl-dev boost-dev libpq-dev curl-dev sqlite-dev && \
    rm -rf /build

RUN adduser -Ds /bin/false pdns pdns

EXPOSE 53/tcp 53/udp

ENTRYPOINT [ "/entrypoint.sh" ]
