# syntax=docker/dockerfile:experimental
FROM ubuntu:xenial as build
LABEL maintainer="Matthias Blaser <mb@emgag.com>"

#
# install varnish build deps
#
RUN \
    --mount=type=cache,target=/var/cache/apt \
    --mount=type=cache,target=/var/lib/apt/lists \
        apt-get update \
        && apt-get install -y --no-install-recommends \
            automake \
            autotools-dev \
            build-essential \
            ca-certificates \
            curl \
            git \
            libedit-dev \
            libgetdns-dev \
            libjemalloc-dev \
            libmhash-dev \
            libncurses-dev \
            libpcre3-dev \
            libtool \
            pkg-config \
            python3 \
            python3-docutils \
            python3-sphinx \
        && apt-get autoremove -y

#
# install varnish
#
ENV VARNISH_VERSION=6.3.2
ENV VARNISH_SHA256SUM=0823744e54a5f0bf7c7871e4c2600e81723bc75e3e42596474a6aa01db87aab8

RUN mkdir -p /usr/local/src && \
    cd /usr/local/src && \
    curl -sfLO http://varnish-cache.org/_downloads/varnish-${VARNISH_VERSION}.tgz && \
    echo "${VARNISH_SHA256SUM} varnish-${VARNISH_VERSION}.tgz" | sha256sum -c - && \
    tar -xzf varnish-${VARNISH_VERSION}.tgz && \
    cd varnish-${VARNISH_VERSION} && \
    ./autogen.sh && \
    ./configure && \
    make install && \
    cd /usr/local/src && \
    rm -rf varnish-*

#
# install stock varnish module library
#
ENV VARNISHMODULES_BRANCH=6.3
ENV VARNISHMODULES_COMMIT=3608d1ec11793de1bc94492b4a2d1b8d48a6ca98

RUN cd /usr/local/src/ && \
    git clone -b ${VARNISHMODULES_BRANCH} https://github.com/varnish/varnish-modules.git && \
    cd varnish-modules && \
    git reset --hard ${VARNISHMODULES_COMMIT} && \
    ./bootstrap && \
    ./configure && \
    make install && \
    cd /usr/local/src && \
    rm -rf varnish-modules && \
    ldconfig


#
# install libvmod-dynamic
#
ENV LIBVMOD_DYNAMIC_BRANCH=master
ENV LIBVMOD_DYNAMIC_COMMIT=88161b3018305915c9cf0d2b45f695da709275da

RUN cd /usr/local/src/ && \
    git clone -b ${LIBVMOD_DYNAMIC_BRANCH} https://github.com/nigoroll/libvmod-dynamic.git && \
    cd libvmod-dynamic && \
    git reset --hard ${LIBVMOD_DYNAMIC_COMMIT} && \
    ./autogen.sh && \
    ./configure && \
    make install && \
    cd /usr/local/src && \
    rm -rf libvmod-dynamic && \
    ldconfig

#
# install libvmod-digest
#
ENV LIBVMOD_DIGEST_BRANCH=6.3
ENV LIBVMOD_DIGEST_COMMIT=1793bea9e9b7c7dce4d8df82397d22ab9fa296f0

RUN cd /usr/local/src/ && \
    git clone -b ${LIBVMOD_DIGEST_BRANCH} https://github.com/varnish/libvmod-digest.git && \
    cd libvmod-digest && \
    git reset --hard ${LIBVMOD_DIGEST_COMMIT} && \
    ./autogen.sh && \
    ./configure && \
    make install && \
    cd /usr/local/src && \
    rm -rf libvmod-digest && \
    ldconfig

#
# install libvmod-querystring
#
ENV LIBVMOD_QUERYSTRING_VERSION=2.0.1
ENV LIBVMOD_QUERYSTRING_SHA256SUM=34540b0fb515bfbf9aaa4154be5372ce5aa8c7050f35f07dc186c85bb7e976c0

RUN cd /usr/local/src/ && \
    curl -sfLO https://github.com/Dridi/libvmod-querystring/releases/download/v${LIBVMOD_QUERYSTRING_VERSION}/vmod-querystring-${LIBVMOD_QUERYSTRING_VERSION}.tar.gz && \
    echo "${LIBVMOD_QUERYSTRING_SHA256SUM} vmod-querystring-${LIBVMOD_QUERYSTRING_VERSION}.tar.gz" | sha256sum -c - && \
    tar -xzf vmod-querystring-${LIBVMOD_QUERYSTRING_VERSION}.tar.gz && \
    cd vmod-querystring-${LIBVMOD_QUERYSTRING_VERSION} && \
    ./configure && \
    make install && \
    cd /usr/local/src && \
    rm -rf vmod-querystring* && \
    ldconfig


FROM alpine
RUN mkdir /dist
COPY --from=build /usr/local/lib/varnish/vmods/*.so /dist
COPY --from=build /usr/local/man/man3/*.3 /dist
WORKDIR /dist
RUN rm *vmod_blob* *vmod_directors* *vmod_purge* *vmod_std* *vmod_vtc* *vmod_unix* *vmod_proxy*
RUN gzip *.3

