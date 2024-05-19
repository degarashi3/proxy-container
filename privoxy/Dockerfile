#
# Dockerfile for privoxy
#

FROM alpine:3

ARG TARGETPLATFORM
ARG GOSU_VERSION=1.16

RUN set -xe \
    && echo ${TARGETPLATFORM} \
    && GOSU_ARCH=$(echo $TARGETPLATFORM | sed -e 's@linux/@@' \
                                              -e 's@amd64@amd64@' \
                                              -e 's@arm64@arm64@' \
                                              -e 's@arm/v7@armhf@') \
    && apk add --no-cache curl bash privoxy \
    && curl -sSL https://github.com/tianon/gosu/releases/download/${GOSU_VERSION}/gosu-${GOSU_ARCH} > /usr/sbin/gosu \
    && chmod +x /usr/sbin/gosu \
    && apk del curl

WORKDIR /etc/privoxy

RUN set -xe \
    && for cfg in *.new; do touch ${cfg%.new}; done \
    && sed -e '/^listen-address/s/127.0.0.1/0.0.0.0/' \
           -e '/^accept-intercepted-requests/s/0/1/' \
           -e '/^enforce-blocks/s/0/1/' \
           -e '/^#debug/s/#//' config.new > config

VOLUME /etc/privoxy

COPY privoxy-adblock.sh /tmp/
RUN chmod +x /tmp/privoxy-adblock.sh

RUN echo "30 2 * * * /bin/bash /tmp/privoxy-adblock.sh -p /etc/privoxy -u http://tofukko.r.ribbon.to/Adblock_Plus_list.txt \
 -u https://raw.githubusercontent.com/k2jp/abp-japanese-filters/master/abpjf.txt > /dev/null 2>&1" > crontab.txt \
&& crontab crontab.txt

EXPOSE 8118

CMD gosu privoxy privoxy --no-daemon /etc/privoxy/config
