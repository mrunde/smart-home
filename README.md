# smart-home

Smart home services running on a Raspberry Pi with Docker Compose.

## Set up the Raspberry Pi

1. Use Raspberry Pi Imager to install a 64-bit Raspberry Pi OS on the Pi. Configure a
   user, network connection, and SSH in the Imager settings, then boot the Pi and
   sign in locally or over SSH.
2. Update the system and install Git:

   ```sh
   sudo apt update
   sudo apt upgrade
   sudo apt install git
   ```
3. Install Docker:

   ```sh
   curl -sSL https://get.docker.com | sh
   ```
4. Start Docker and allow your user to run it without `sudo`:

   ```sh
   sudo systemctl enable --now docker
   sudo usermod -aG docker "$USER"
   ```

   Log out and back in so that the new group membership takes effect. Verify with
   `docker version` and `docker compose version`. Membership in the `docker`
   group grants root-equivalent privileges; only grant it to trusted users.

## Prepare the deployment

Clone the repository as your normal user, not as root. The Compose file and
service directories are in the repository root:

```sh
cd ~
git clone https://github.com/mrunde/smart-home.git
cd smart-home
cp .env.example .env
cp mosquitto/acl_file.example mosquitto/acl_file
mkdir -p home-assistant/config home-assistant/media music-assistant/data \
  esphome/config zigbee2mqtt/data mosquitto/data mosquitto/log postgres/data
```

Edit `.env`: set a strong PostgreSQL password and MQTT password, and set
`ZIGBEE_DEVICE` to the actual adapter path shown by `ls -l /dev/serial/by-id/`.
The supplied ACL expects `MOSQUITTO_USER=homeassistant`; update the ACL and
the username in the `mosquitto_passwd` command below if you choose another
username. Use distinct passwords for PostgreSQL and MQTT.

Create the MQTT password file interactively, using the value of
`MOSQUITTO_PASSWORD` from `.env` for **both** accounts:

```sh
docker compose run --rm --user "$(id -u):$(id -g)" mosquitto-setup \
  -c /mosquitto/config/password_file homeassistant

docker compose run --rm --user "$(id -u):$(id -g)" mosquitto-setup \
  /mosquitto/config/password_file zigbee2mqtt

chmod 644 mosquitto/password_file
chmod 700 mosquitto
chmod 600 .env
```

The password file contains password hashes and must be readable by the broker
inside its container. Keep it and `.env` private and never commit them. The
`-c` command replaces an existing password file; do not
repeat it unless you intend to recreate the accounts.

## Start the services

For a **new** Zigbee2MQTT installation, seed its data directory once from
the tracked configuration. The command leaves an existing configuration intact:

```sh
docker compose run --rm --no-deps --entrypoint sh zigbee2mqtt \
  -c 'test -f /app/data/configuration.yaml || cp /seed/configuration.yaml /app/data/configuration.yaml'
docker compose config --quiet
docker compose up -d
docker compose ps
docker compose logs --tail=100
```

The seed command requires the Zigbee adapter configured in `.env` to be
connected. Once seeded, Zigbee2MQTT manages its own configuration in the
data directory; changing `zigbee2mqtt/configuration.yaml` in the repository does not
automatically overwrite that live configuration.

Configure Home Assistant's MQTT integration to connect to `127.0.0.1:1883`
using the configured `homeassistant` account and MQTT password. Zigbee2MQTT
uses the `zigbee2mqtt` account over the Compose network. MQTT and PostgreSQL
are published only on the Pi's loopback interface, so remote clients cannot
connect directly.

## Updates

Review release notes and back up your data before updating images:

```sh
git pull
docker compose pull
docker compose up -d
docker compose ps
```

## Restore a Home Assistant Backup

Copy the backup file (e.g. `backup.tar`) from your Windows computer to the user's directory on the Raspberry Pi
and from there to the `homeassistant` directory.

On your Windows computer:
```bat
scp .\backup.tar <<USERNAME>>@raspberrypi.local:~/
```

On the Raspberry Pi:
```sh
sudo cp /home/<<USERNAME>>/backup.tar ./home-assistant/backup.tar
```

Now go to the `/opt/docker/home-assistant` directory and extract the backup into the `config` directory. After
that you can restart the Home Assistant Docker container.

```sh
sudo tar -xOf backup.tar homeassistant.tar.gz | sudo tar --strip-components=1 -zxf - -C config
```
