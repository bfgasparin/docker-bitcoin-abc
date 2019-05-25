FROM ubuntu:xenial
MAINTAINER Bruno Gasparin <bfgasparin@gmail.com>

ARG USER_ID
ARG GROUP_ID

ENV HOME /bitcoin

# add user with specified (or default) user/group ids
ENV USER_ID ${USER_ID:-1000}
ENV GROUP_ID ${GROUP_ID:-1000}

# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN groupadd -g ${GROUP_ID} bitcoin-abc \
	&& useradd -u ${USER_ID} -g bitcoin-abc -s /bin/bash -m -d /bitcoin bitcoin-abc

# grab gosu for easy step-down from root
ENV GOSU_VERSION 1.7
RUN set -x \
	&& apt-get update && apt-get install -y --no-install-recommends \
		ca-certificates \
		wget \
	&& wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
	&& wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
	&& gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
	&& rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc \
	&& chmod +x /usr/local/bin/gosu \
	&& gosu nobody true \
	&& apt-get purge -y \
		ca-certificates \
		wget \
	&& apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV BITCOIN_ABC_VERSION="0.19.5"
ENV BITCOIN_ABC_SHASUM="6b95bc75dd0f40eb86076bae002a87a1878b4f12614dfeeefa888ea4fdaebf8a  bitcoin-abc-${BITCOIN_ABC_VERSION}-x86_64-linux-gnu.tar.gz"
ENV BITCOIN_ABC_PREFIX=/opt/bitcoin-abc-${BITCOIN_ABC_VERSION}
ENV BITCOIN_ABC_DATA=/bitcoin/.bitcoin
ENV PATH=/bitcoin/bitcoin-abc-${BITCOIN_ABC_VERSION}/bin:$PATH

RUN set -x \
	&& apt-get update && apt-get install -y --no-install-recommends \
		ca-certificates \
                gnupg \
                unzip \
		curl \
        && curl -SLO https://download.bitcoinabc.org/${BITCOIN_ABC_VERSION}/linux/bitcoin-abc-${BITCOIN_ABC_VERSION}-x86_64-linux-gnu.tar.gz \
        && echo "${BITCOIN_ABC_SHASUM}" | sha256sum -c \
        && tar -xzf *.tar.gz -C /bitcoin \
        && rm *.tar.gz \
	&& apt-get purge -y \
		ca-certificates \
		curl \
	&& apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ADD ./bin /usr/local/bin

VOLUME ["/bitcoin"]

EXPOSE 8332 8333 18332 18333

WORKDIR /bitcoin

COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]

CMD ["btc_oneshot"]
