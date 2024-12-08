FROM debian:bookworm-slim AS build-stage

MAINTAINER Ivo Smid <ivo.smid@gmail.com>

ARG BUILD_OPTIONS

RUN apt-get update \
 && apt-get install -y apt-transport-https \
 && apt-get install -y --no-install-recommends \
     # build
     git \
     pkg-config \
     libtool \
     libc6-dev \
     autoconf \
     automake \
     make \
     build-essential \
     ca-certificates \
     # dependencies
     libcurl4-openssl-dev \
     libncurses5-dev \
     libusb-1.0-0-dev \
     libudev-dev \
     uthash-dev \
     libjansson-dev \
     zlib1g-dev \
 && rm -rf /var/lib/apt/lists/* \
 && rm -rf /tmp/downloaded_packages/ /tmp/*.rds

RUN mkdir -p /opt/cgminer-build

RUN git clone https://github.com/ckolivas/cgminer.git /opt/cgminer-build

WORKDIR /opt/cgminer-build

RUN ./autogen.sh \
 && CFLAGS="-O2 -Wall -march=native -fcommon" ./configure ${BUILD_OPTIONS} \
 && make

# ------------------------------------------------ RUN ------------------------------------------------

FROM debian:bookworm-slim

ENV TERM linux
ENV POOL_URL stratum+tcp://de.ckpool.org:443
ENV POOL_USER x
ENV POOL_PASSWORD x
ENV GEKKO_2PAC_FREQ 200
ENV GEKKO_NEWPAC_FREQ 200
ENV SUGGEST_DIFF 32
ENV API_PORT 4028

RUN apt-get update \
 && apt-get install -y apt-transport-https \
 && apt-get install -y --no-install-recommends \
     # dependencies
     libcurl4-openssl-dev \
     libncurses5-dev \
     libusb-1.0-0-dev \
     libudev-dev \
     uthash-dev \
     libjansson-dev \
 && rm -rf /var/lib/apt/lists/* \
 && rm -rf /tmp/downloaded_packages/ /tmp/*.rds

RUN mkdir -p /opt/cgminer

WORKDIR /opt/cgminer

COPY --from=build-stage /opt/cgminer-build/cgminer .

EXPOSE $API_PORT

ENTRYPOINT ./cgminer -o $POOL_URL -u $POOL_USER -p $POOL_PASSWORD --gekko-2pac-freq $GEKKO_2PAC_FREQ --gekko-newpac-freq $GEKKO_NEWPAC_FREQ --suggest-diff $SUGGEST_DIFF --api-listen --api-network --api-port $API_PORT --text-only

# docker run --privileged 865b48061d9e ./cgminer -n