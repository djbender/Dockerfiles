# syntax=docker/dockerfile:1.4
FROM ubuntu:jammy

ARG APT_MIRROR
ENV OPENSSL_VERSION=1.1.1q

RUN <<EOT
#!/usr/bin/env bash
  set -exu
  [ ! -z $APT_MIRROR ] \
    && sed -i "s/archive.ubuntu.com/$APT_MIRROR/" /etc/apt/sources.list
  apt-get update
  apt-get install --no-install-recommends --yes \
    ca-certificates \
    curl \
    build-essential \
    zlib1g-dev
  curl -s https://www.openssl.org/source/openssl-1.1.1q.tar.gz | tar -C /usr/local/src -zx
  pushd /usr/local/src/openssl-1.1.1q
  ./config --prefix=/usr/local/ssl --openssldir=/usr/local/ssl shared zlib
  make -j $(nproc)
  make -j $(nproc) test
  make install
  popd
  echo "/usr/local/ssl/lib" > /etc/ld.so.conf.d/openssl-1.1.1q.conf
  ldconfig -v | grep /usr/local/ssl/lib
  echo "PATH=/usr/local/ssl/bin:$PATH" > /etc/environment
  source /etc/environment
  apt-get remove --yes \
    build-essential \
    curl \
    zlib1g-dev
  apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false
  rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /var/cache/*

  # verify
  [ "$(command -v openssl)" = '/usr/local/ssl/bin/openssl' ]
  openssl version -a
EOT

RUN <<EOT
#!/usr/bin/env bash
  set -exu
  apt-get update
  apt-get install --no-install-recommends --yes \
    ca-certificates \
    curl \
    make \
    python3-cryptography\
    xz-utils

  mkdir /usr/local/src/ca-certificates_20211016
  curl -sL https://launchpad.net/ubuntu/+archive/primary/+sourcefiles/ca-certificates/20211016/ca-certificates_20211016.tar.xz | tar -Jx -C /usr/local/src/ca-certificates_20211016 --strip-components=1
  apt-get remove --yes ca-certificates

  pushd /usr/local/src/ca-certificates_20211016
  sed -i 's/mkdir/mkdir -p/' Makefile
  make -j $(nproc)
  make install
  popd

  # verify
  curl -sI https://github.com

  apt-get remove --yes \
    curl \
    make \
    python3-cryptography \
    xz-utils
  apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false
  rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /var/cache/*
EOT
