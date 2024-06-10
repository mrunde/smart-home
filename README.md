# smart-home

Smart Home application running on a Raspberry Pi

## Setup

This describes the initial setup of the project.

- Update and upgrade the Raspberry Pi

    ```
    sudo apt update
    sudo apt upgrade
    ```

- Install Docker

    ```
    curl -sSL https://get.docker.com | sh
  
    docker --version
    ```

- Add your user to the Docker group

    ```
    sudo usermod -aG docker $USER
    ```

- Logout to apply the changes to the user

    ```
    logout
    ```

## Run with Docker Compose

After the setup you have to copy the `docker-compose.yaml`[docker/docker-compose.yaml] into the `/opt/docker` directory of the Raspberry Pi.

Also create the following directories:

```
/opt/docker/homeassistant
/opt/docker/homeassistant/images

/opt/docker/pihole

/opt/docker/postgresql
/opt/docker/postgresql/data

/opt/docker/pgadmin
```

Finally, you have to copy the `credentials.env` files into their respective directories (e.g. `/opt/docker/postgresql`).

Now you can start the Docker containers with the following command (`-d` for detached mode):

```
docker compose up -d
```
