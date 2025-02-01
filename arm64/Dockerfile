ARG BASE_IMAGE=openwrt/rootfs:aarch64_cortex-a53-v21.02.5
FROM ${BASE_IMAGE}

LABEL maintainer="DianQK <dianqk@icloud.com>"

ENV UU_LAN_IPADDR=
ENV UU_LAN_GATEWAY=
ENV UU_LAN_NETMASK="255.255.255.0"
ENV UU_LAN_DNS="119.29.29.29"

USER root

RUN mkdir -p /var/lock && \
    echo 'src/gz openwrt_core https://downloads.openwrt.org/releases/21.02.5/targets/armvirt/64/packages' > /etc/opkg/distfeeds.conf && \
    echo 'src/gz openwrt_base https://downloads.openwrt.org/releases/21.02.5/packages/aarch64_cortex-a53/base' >> /etc/opkg/distfeeds.conf && \
    echo 'src/gz openwrt_luci https://downloads.openwrt.org/releases/21.02.5/packages/aarch64_cortex-a53/luci' >> /etc/opkg/distfeeds.conf && \
    echo 'src/gz openwrt_packages https://downloads.openwrt.org/releases/21.02.5/packages/aarch64_cortex-a53/packages' >> /etc/opkg/distfeeds.conf && \
    echo 'src/gz openwrt_routing https://downloads.openwrt.org/releases/21.02.5/packages/aarch64_cortex-a53/routing' >> /etc/opkg/distfeeds.conf && \
    echo 'src/gz openwrt_telephony https://downloads.openwrt.org/releases/21.02.5/packages/aarch64_cortex-a53/telephony' >> /etc/opkg/distfeeds.conf

RUN opkg update && \
opkg list-installed | grep -q libustream-openssl || opkg install libustream-openssl && \
opkg list-installed | grep -q ca-bundle || opkg install ca-bundle && \
opkg list-installed | grep -q kmod-tun || opkg install kmod-tun && \
rm -rf /var/opkg-lists

RUN opkg update && \
    if ! opkg list-installed | grep -q libustream-openssl; then opkg install libustream-openssl || true; fi && \
    if ! opkg list-installed | grep -q ca-bundle; then opkg install ca-bundle || true; fi && \
    if ! opkg list-installed | grep -q kmod-tun; then opkg install kmod-tun || true; fi && \
    rm -rf /var/opkg-lists


COPY uu_prepare /etc/init.d/uu_prepare
RUN chmod +x /etc/init.d/uu_prepare \
    && /etc/init.d/uu_prepare enable \
    && /etc/init.d/odhcpd disable \
    && /etc/init.d/firewall disable \
    && /etc/init.d/uhttpd disable \
    && /etc/init.d/dropbear disable

CMD ["/sbin/init"]
