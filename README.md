# security-perimeter
Build a security perimeter around your home network and guard the important resources at your home

Most home networks have an WiFi Access Point and a Cable Modem / DSL to connect to the Internet. The goal of this project is to setup useful services in the home network and access them securely. Currently these features are supported.

    Password Manager (Vaultwarden)
    Ad blocker (Pihole)
    NAT (iptables)
    Remote Access (wireguard)
    
# Docker compose details


* A separate network (home) is configured with a prefix of 172.16.0.0/16 rather than using the docker’s default network 172.17.0.0/16. Pi-hole container running a dhcp server needs a static IP address that can be used by dhcp-helper to forward the DHCP packets. Trying to assign an IP address from default network does not seem to work

* Dhcp-helper is running in network mode listening on ports 67/68 receiving DHCP packets on the LAN interface and forwarding them to pihole container. Pihole is listening on all interfaces (DNSMASQ_LISTENING: all)

* There are two sets of DNS settings:
    * dns : This tells the container how to resolve DNS queries. Specifying 127.0.0.1 is a requirement which makes locally originating dns requests from within the container go through pi-hole. There is a health check done by pihole by resolving pi-hole (sudo tcpdump -i lo port 53 inside the container will reveal this) and it needs to resolve successfully. See here: https://github.com/pi-hole/docker-pi-hole/issues/431
    * DNS1 and DNS2 are used by pihole as upstream dns servers. This can be changed later from the UI to a different set of servers.
    * DHCP settings specify the address pool and the router. The router address should match the ServerIP the security gateway will be the default router
    * Create a file 07-dhcp-options under etc-dnsmasq.d with the following contents: dhcp-option=option:dns-server,192.168.100.1 (This is required so that the right dns server is returned in DHCP options; otherwise the container address is returned)
    * Caddy settings assumes duckdns and follow the instructions given here: https://github.com/dani-garcia/vaultwarden/wiki/Using-Docker-Compose
