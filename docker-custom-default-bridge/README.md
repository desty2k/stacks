# Docker stack with custom default bridge

This example shows how to create custom default bridge network and use it in `docker-compose.yml` file.

## Why use custom bridge network?
- Better isolation
- Assigning static IP to containers
- Automatic DNS resolution between containers
- Each user-defined network creates separate virtual interface

## Installation
Creates two whoami containers serving on port `80` and `81`.

1. Update timezone in `.env`
2. Run compose file
    ```shell
    docker-compose up -d
    ```
3. Check if new network was created by Docker
    ```shell
    docker network ls
    ```
4. Open `SERVER_IP:80` and `SERVER_IP:81` in your web browser to check if services are running

## Potential problems

- error `network ... not found` when running `docker-compose`. 

  Use `--force-recreate` flag
  ```shell
  docker compose up -d --force-recreate
  ```
  
- removing network
  
  Use `docker network rm NAME`.
  ```shell
  docker network rm internal
  ```


### Sources
- https://docs.docker.com/network/bridge/#differences-between-user-defined-bridges-and-the-default-bridge
