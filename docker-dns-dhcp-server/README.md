# Pi-Hole DNS + DHCP in bridge mode

## Installation

### Variables:
**Key**|**Value**|**Example**
:-----:|:-----:|:-----:
SERVER_IP|IP address of your server in LAN|192.168.10.2 __or__ 10.10.0.2

### Configuration

1. Copy `pihole` directory contents to `/docker` directory on your server
    ```shell
    mkdir /docker
    cp -r pihole/* /docker
    ```
2. Update Pi-Hole admin password in `/docker/secrets/pihole_admin_password`
3. Update `SERVER_IP` and `TZ` variables in `/docker/.env` file
4. Start containers using `docker-compose`
    ```shell
    cd /docker
    docker-compose up -d
    ```
5. Open `http://SERVER_IP/admin` in web browser
6. Log in to admin panel using admin password
7. On left toolbar click `Settings`
8. In top toolbar select `DNS`
9. In `Interface listening behavior` section check `Listen on all interfaces, permit all origins` option
10. Save changes using button at the bottom of page
11. In top toolbar select `DHCP`
12. In `DHCP Settings` check `DHCP server enabled`. Update IP range and gateway IP.
13. Save changes using button at the bottom of page

### Router configuration
Disable DHCP server on router.

### Testing
Wait for devices to update their network configuration from new DHCP server or set DNS statically.
On `Windows` use `nslookup` to check if DNS server is responding.
```shell
C:\Users\user>nslookup duckduckgo.com
Server:  UnKnown
Address:  192.168.10.2  # should be equal to SERVER_IP

Non-authoritative answer:
Name:    duckduckgo.com
Address:  40.114.177.156
```
On linux use `dig` to test DNS server.
```shell
user@server:/docker# dig duckduckgo.com
...
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58716
...
;; Query time: 1 msec
;; SERVER: 192.168.10.2#53(192.168.10.2)  # should be equal to SERVER_IP
...
```

## Sources
https://github.com/pi-hole/docker-pi-hole

https://docs.pi-hole.net/docker/dhcp

https://github.com/homeall/dhcphelper
