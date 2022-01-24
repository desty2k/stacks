# Run Tasmota without Internet access
Tasmota requires constant internet connection by design. 
To run Tasmota devices with local network access only we need to run our own NTP server.
`SERVER_IP` is local IP of your server in local network.

## Installation

1. Update `TZ` variable in `.env` file
2. Start containers using `docker compose`
    ```shell
    docker compose up -d
    ```
3. Set MQTT username and password in `appdata/mosquitto/passwd` by using `docker exec`.
Replace `USERNAME` and `PASSWORD` with appropriate values.
    ```shell
    docker exec mosquitto -it /bin/bash -c mosquitto_passwd /mosquitto/passwd USERNAME PASSWORD
    ```
4. Restart container using `docker restart`
   ```shell
   docker restart mosquitto
   ```
5. Open `SERVER_IP:8080` in your browser
6. Scan for Tasmota devices in your network
7. Execute following command on all devices
```shell
NtpServer1 SERVER_IP
```

## Sources
https://github.com/arendst/Tasmota/issues/12304
