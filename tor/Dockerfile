FROM alpine:3.4
MAINTAINER Wonderfall <wonderfall@schrodinger.io>

ARG TOR_VERSION=0.2.8.9
ARG ARM_VERSION=1.4.5.0

ARG GPG_Mathewson="B35B F85B F194 89D0 4E28  C33C 2119 4EBB 1657 33EA"
ARG GPG_Johnson="6827 8CC5 DD2D 1E85 C4E4  5AD9 0445 B7AB 9ABB EEC6"

ENV TERM=xterm \
    UID=991 GID=991

RUN echo "@community https://nl.alpinelinux.org/alpine/v3.4/community" >> /etc/apk/repositories \
 && BUILD_DEPS=" \
    libevent-dev \
    openssl-dev \
    build-base \
    gnupg \
    ca-certificates" \
 && apk -U add \
    ${BUILD_DEPS} \
    su-exec \
    tini@community \
    python \
    libevent \
    openssl \
 && cd /tmp \
 && TOR_TARBALL="tor-${TOR_VERSION}.tar.gz" \
 && wget -q https://www.torproject.org/dist/${TOR_TARBALL} \
 && echo "Verifying ${TOR_TARBALL} using GPG..." \
 && wget -q https://www.torproject.org/dist/${TOR_TARBALL}.asc \
 && gpg --keyserver keys.gnupg.net --recv-keys 0x165733EA \
 && FINGERPRINT="$(LANG=C gpg --verify ${TOR_TARBALL}.asc ${TOR_TARBALL} 2>&1 \
  | sed -n "s#Primary key fingerprint: \(.*\)#\1#p")" \
 && if [ -z "${FINGERPRINT}" ]; then echo "Warning! Invalid GPG signature!" && exit 1; fi \
 && if [ "${FINGERPRINT}" != "${GPG_Mathewson}" ]; then echo "Warning! Wrong GPG fingerprint!" && exit 1; fi \
 && echo "All seems good, now unpacking ${TOR_TARBALL}..." \
 && tar xzf ${TOR_TARBALL} && cd tor-${TOR_VERSION} \
 && ./configure --disable-asciidoc && make && make install \
 && cd /tmp \
 && ARM_TARBALL="arm-${ARM_VERSION}.tar.bz2" \
 && wget -q https://www.atagar.com/arm/resources/static/${ARM_TARBALL}  \
 && echo "Verifying ${ARM_TARBALL}..." \
 && wget -q https://www.atagar.com/arm/resources/static/${ARM_TARBALL}.asc \
 && gpg --keyserver pgp.mit.edu --recv-keys 0x9ABBEEC6 \
 && FINGERPRINT="$(LANG=C gpg --verify ${ARM_TARBALL}.asc ${ARM_TARBALL} 2>&1 \
  | sed -n "s#Primary key fingerprint: \(.*\)#\1#p")" \
 && if [ -z "${FINGERPRINT}" ]; then echo "Warning! Invalid GPG signature!" && exit 1; fi \
 && if [ "${FINGERPRINT}" != "${GPG_Johnson}" ]; then echo "Warning! Wrong GPG fingerprint!" && exit 1; fi \
 && echo "All seems good, now unpacking ${ARM_TARBALL}..." \
 && tar xjf /tmp/${ARM_TARBALL} && cd arm && ./install \
 && apk del ${BUILD_DEPS} \
 && rm -rf /var/cache/apk/* /tmp/* /root/.gnupg

COPY run.sh /usr/local/bin/run.sh

RUN chmod +x /usr/local/bin/run.sh

VOLUME /tor/config /tor/data

EXPOSE 9001 9030

ENTRYPOINT ["run.sh"]
