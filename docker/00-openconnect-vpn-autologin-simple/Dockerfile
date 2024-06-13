# This dockerfile requires the openconnect-passwd.txt file in folder. It may be any pseudofile if need.
LABEL version=0.2 vpn_type=openconnect
FROM golang:latest AS proxy-build
# Build SOCKS5 proxy for a controlled access to VPN (exposed by PROXY_PORT env)

WORKDIR /root

# d0347549a4ec308b67f39d24d76691c4d4b1da6a is the last builable commit for now (2024-06-13.
# See https://github.com/serjs/socks5-server/issues/61 .
ARG proxy_branch=master
# TODO: Change to main/master after resolving issue.
ARG proxy_commit=d0347549a4ec308b67f39d24d76691c4d4b1da6a
RUN git clone https://github.com/serjs/socks5-server && cd socks5-server && git checkout $proxy_commit && go build .


FROM fedora:latest AS fedora-base
# Simple All update/upgrade/install deps step.
# crypto-policies-scripts is the helper for enabling legacy OpenSSL algorithms (it may be required for some AnyConnect hosts)
# See https://github.com/dotnet/dotnet-docker/issues/4332 for details.
RUN dnf upgrade --refresh -y && dnf install openconnect crypto-policies-scripts -y && update-crypto-policies --set LEGACY && dnf clean all

# Our container API for connecting
ARG proxy_port
ARG vpn_host
ARG vpn_user
ARG vpn_authgroup
ENV PROXY_PORT=$proxy_port
ENV VPN_HOST=$vpn_host
ENV VPN_USER=$vpn_user
ENV VPN_AUTHGROUP=$vpn_authgroup

WORKDIR /root
# Copy your password here (insecure, TODO: fix it??)
COPY ./openconnect-passwd.txt .
# Copy golang socks5 proxy binary from another image
COPY --from=proxy-build /root/socks5-server/socks5-server .

# Expose port for SOCKS5 clients
EXPOSE $PROXY_PORT

# Run openconnect with our password and run proxy.
CMD cat openconnect-passwd.txt | openconnect --background -u $VPN_USER --authgroup $VPN_AUTHGROUP $VPN_HOST && ./socks5-server
# CMD openconnect -u $VPN_USER --script-tun --script \
#    "ocproxy -D $PROXY_PORT -g -k 36000 " \
#    $VPN_HOST 

