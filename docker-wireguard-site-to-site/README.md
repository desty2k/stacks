# Contenerized Wireguard Site-to-Site

## Installation

### Variables:
**Key**|**Value**|**Example**
:-----:|:-----:|:-----:
SERVER1\_PUBLIC\_ADDRESS|public IP or DDNS or domain pointing to server1|24.151.98.12 __or__ server1.duckdns.org __or__ server1domain.com
SERVER2\_PUBLIC\_ADDRESS|public IP or DDNS or domain pointing to server2|69.211.63.75 __or__ server2.duckdns.org __or__ server2domain.com
SERVER1\_LOCAL\_NETWORK|server1 local network address in CIDR format|192.168.1.0/24
SERVER2\_LOCAL\_NETWORK|server2 local network address in CIDR format|192.168.2.0/24
SERVER1\_LOCAL\_IP|local IP of server1, must be in SERVER1\_LOCAL\_NETWORK|192.168.1.100
SERVER2\_LOCAL\_IP|local IP of server2, must be in SERVER2\_LOCAL\_NETWORK|192.168.2.200

### Servers configuration
1. Copy `server1` contents to `/docker` directory on first server
2. Copy `server2` contents to `/docker` directory on second server
3. Update timezone in both `/docker/.env` files
4. On `server1` start containers using `docker compose`
    ```shell
    cd /docker
    docker-compose up -d
    ```
5. Wait for Docker to download image and start container
6. Open container shell using `docker exec`
    ```shell
    docker exec -it wireguard-sts /bin/bash
    ```
7. Generate two pairs of private and public key
    ```shell
    wg genkey | tee privatekey1 | wg pubkey > publickey1
    wg genkey | tee privatekey2 | wg pubkey > publickey2
    ```
8. Save contents of `privatekey1`, `privatekey1`, `privatekey2` and `publickey2` to your text editor
9. Remove created files
    ```shell
    rm privatekey1 privatekey1 privatekey2 publickey2
    ``` 
10. Exit shell and stop container
    ```shell
    exit
    docker container stop wireguard-sts
    ```
11. On `server1` update `/docker/appdata/wireguard-sts/wg0.conf`
    * `SERVER1_PRIVATE_KEY` -> `privatekey1`
    * `SERVER2_PUBLIC_KEY` -> `publickey2`
    * `SERVER2_LOCAL_NETWORK`
    * `SERVER2_PUBLIC_ADDRESS`
    

12. On `server2` update `/docker/appdata/wireguard-sts/wg0.conf`
    * `SERVER2_PRIVATE_KEY` -> `privatekey2`
    * `SERVER1_PUBLIC_KEY` -> `publickey1`
    * `SERVER1_LOCAL_NETWORK`
    * `SERVER1_PUBLIC_ADDRESS`
    

13. On `server1` add following lines to `/etc/dhcpcd.exit-hook` file (create file if it does not exist). 
    Replace variables with the appropriate values.
    ```shell
    ip route replace SERVER2_LOCAL_NETWORK via 172.10.0.100 src SERVER1_LOCAL_IP
    iptables -I FORWARD 1 -s SERVER1_LOCAL_NETWORK -i eth0 -d SERVER2_LOCAL_NETWORK -j ACCEPT
    ```

14. On `server2` add following lines to `/etc/dhcpcd.exit-hook` file (create file if it does not exist).
    Replace variables with the appropriate values.
    ```shell
    ip route replace SERVER1_LOCAL_NETWORK via 172.20.0.100 src SERVER2_LOCAL_IP
    iptables -I FORWARD 1 -s SERVER2_LOCAL_NETWORK -i eth0 -d SERVER1_LOCAL_NETWORK -j ACCEPT
    ```
15. On both servers change working directory to `/docker` and run `docker-compose`
    ```shell
    cd /docker
    docker-compose up -d
    ```
16. Wait for Docker to download images and start containers
17. Reboot both servers

### Routers configuration
1. On `router1` forward port `61820` to `SERVER1_LOCAL_IP`
2. On `router2` forward port `61820` to `SERVER2_LOCAL_IP`
3. On `router1` add static routing to SERVER2_LOCAL_NETWORK via SERVER1_LOCAL_IP
4. On `router2` add static routing to SERVER1_LOCAL_NETWORK via SERVER2_LOCAL_IP

### Testing
1. Ping `router1` from `SERVER1_LOCAL_IP` - __works__
2. Ping `router2` from `SERVER2_LOCAL_IP` - __works__
3. Ping `SERVER2_LOCAL_IP` from `SERVER1_LOCAL_IP` - __works__
4. Ping `SERVER1_LOCAL_IP` from `SERVER2_LOCAL_IP` - __works__
5. Ping `router1` from `SERVER2_LOCAL_IP` - __works__
6. Ping `router2` from `SERVER1_LOCAL_IP` - __works__
7. Ping any host in `SERVER1_LOCAL_NETWORK` from `SERVER2_LOCAL_IP` - __works__
8. Ping any host in `SERVER2_LOCAL_NETWORK` from `SERVER1_LOCAL_IP` - __works__
9. Ping any host in `SERVER2_LOCAL_NETWORK` from any host in `SERVER1_LOCAL_NETWORK` - __works__
10. Ping any host in `SERVER1_LOCAL_NETWORK` from any host in `SERVER2_LOCAL_NETWORK` - __works__

### Potential problems
When using DDNS Wireguard will not restart tunnel if peer's IP address change. 
Solution is to use [service, that will periodically reresolve endpoint DNS](https://wiki.archlinux.org/title/WireGuard#Endpoint_with_changing_IP). 


## Sources
https://serverfault.com/questions/1066557/site2site-wireguard-with-docker-routing-problems
https://serverfault.com/questions/201972/linux-gateway-not-forwarding-packets

