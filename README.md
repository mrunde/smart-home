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

After the setup you have to copy the [`docker-compose.yaml`](Docker/docker-compose.yaml) into the `/opt/docker` 
directory of the Raspberry Pi.

Also create the following directories:

```
/opt/docker/home-assistant/config
/opt/docker/home-assistant/media

/opt/docker/zigbee2mqtt/data

/opt/docker/mosquitto/config
/opt/docker/mosquitto/data
/opt/docker/mosquitto/log

/opt/docker/postgres

/opt/docker/pgadmin
```

Finally, you have to copy the `credentials.env` and `mosquitto.conf` files into their respective directories (e.g. 
`/opt/docker/postgres`).

### Mosquitto Configuration

The `mosquitto.conf` file is the [default configuration](https://raw.githubusercontent.com/eclipse/mosquitto/master/mosquitto.conf) only changed for the following properties:

```
# Listen on port 1883 on all IPv4 interfaces
listener 1883
socket_domain ipv4

# Save the in-memory database to disk
persistence true
persistence_location /mosquitto/data/

# Log to stderr and logfile
log_dest stderr
log_dest file /mosquitto/log/mosquitto.log

# Disable authentication
allow_anonymous true
```

Now you can start the Docker containers with the following command (`-d` for detached mode):

```
docker compose up -d
```

## Update the Home Assistant Docker Container

- Check the current images:
  ```
  docker images
  ```

- Get the container ID:
  ```
  docker ps
  ```

- Stop the Home Assistant container:
  ```
  docker stop <container_id>
  ```

- Remove the container and image:
  ```
  docker rm <container_id>
  docker rmi <image_id>
  ```

- Pull the latest image:
  ```
  docker pull homeassistant/home-assistant:latest
  ```

- Start the updated container:
  ```
  docker compose up -d
  ```

## Restore a Home Assistant Backup

- Copy the backup file (e.g. `backup.tar`) from your Windows computer to the user's directory on the Raspberry Pi 
  and from there to the `homeassistant` directory.

  On your Windows computer:
  ```
  scp .\backup.tar <<USERNAME>>@raspberrypi.local:~/
  ```
  
  On the Raspberry Pi:
  ```
  sudo cp /home/<<USERNAME>>/backup.tar /opt/docker/home-assistant/backup.tar
  ```

- Now go to the `/opt/docker/home-assistant` directory and extract the backup into the `config` directory. After 
  that you can (re)start the Home Assistant Docker container.

  ```
  sudo tar -xOf backup.tar homeassistant.tar.gz | sudo tar --strip-components=1 -zxf - -C config
  ```
