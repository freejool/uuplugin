ARG BASE_IMAGE=openwrt/rootfs:aarch64_generic-21.02.5
FROM ${BASE_IMAGE}

LABEL maintainer="DianQK <dianqk@icloud.com>"

ENV UU_LAN_IPADDR=
ENV UU_LAN_GATEWAY=
ENV UU_LAN_NETMASK="255.255.255.0"
ENV UU_LAN_DNS="119.29.29.29"

USER root

RUN set -ex \
    && mkdir -p /var/lock \
    && opkg update \
    && opkg install \
        libustream-mbedtls \
        ca-certificates \
        kmod-tun \
    && rm -rf /var/opkg-lists

COPY uu_prepare /etc/init.d/uu_prepare
RUN chmod +x /etc/init.d/uu_prepare \
    && /etc/init.d/uu_prepare enable \
    && /etc/init.d/odhcpd disable \
    && /etc/init.d/firewall disable \
    && /etc/init.d/uhttpd disable \
    && /etc/init.d/dropbear disable

CMD ["/sbin/init"]
