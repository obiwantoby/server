# Home Server scripts
Home Server Files

## General Server Setup

**Debian 11**
**Global Unit Files** for systemctl located in /etc/systemd/system/
**User Unit Files** for systemctl --user located in
**NVIDIA MEALLNOX-4lx**

apt-get install pve-headers-$(uname -r)

```bash
sudo apt install mstflint gawk
NEW_FIRMWARE_BIN="CHANGE ME TO THE PATH TO BIN FILE"
# Assume we have only one ConnectX-card and we want to flash only that card
PCI_ID=$(sudo lspci | gawk '($0 ~ /ConnectX/ && $1 ~ /\.0$/){print $1}' | head -n 1)
CARD_MODEL=$(sudo lspci | gawk '($0 ~ /ConnectX/ && $1 ~ /\.0$/){print gensub(/[\]\[]/, "", "g", $8$9)}' | head -n1)
BACKUP_DIR="${CARD_NAME}_${PCI_ID}_backup"
mkdir -p "${BACKUP_DIR}"
sudo mstflint -d "${PCI_ID}" query full > "${BACKUP_DIR}"/full_query.txt
sudo mstflint -d "${PCI_ID}" hw query > "${BACKUP_DIR}"/hw_query.txt
sudo mstflint -d "${PCI_ID}" ri "${BACKUP_DIR}"/orig_firmware.bin
sudo mstflint -d "${PCI_ID}" dc "${BACKUP_DIR}"/orig_firmware.ini
sudo mstflint -d "${PCI_ID}" -i "${NEW_FIRMWARE_BIN}" -allow_psid_change burn
sudo mstfwreset -d "${PCI_ID}" reset
```
### Docker Compose File

```
/data/docker/docker-compose.yml
```

### Planetary Annhilation Dedicated Server

Documentation for the server is [here](https://planetaryannihilation.com/guides/hosting-a-local-server/).

### Satisfactory Dedicated Server

Documentation for the server is [here](https://satisfactory.fandom.com/wiki/Dedicated_servers).

### VTT Tabletop Dedicated Server

All data is stored in 

```
/data/dungeondsanddragons/foundrydata
```
It is analogous to /data/

Documentation for the server is [here](https://foundryvtt.com/article/installation/).

https://github.com/felddy/foundryvtt-docker

### Minecraft

Documentation for the server is [here](https://foundryvtt.com/article/installation/).

### Backup

/usr/local/sbin/cron/backup.sh

/mnt/syn_nas --> NFS share on NAS

```

ip:/volume1/Backups /mnt/backup nfs defaults 0 0
```
### Certbot

In my configuration, I used cloudflare DNS, so I created a API scoped to Zone:DNS:Edit for the dns-01 challenge. 

/root/cred.ini

```
# Cloudflare API token used by Certbot
dns_cloudflare_api_token = 0123456789abcdef0123456789abcdef01234567
```

```
certbot:
                image: certbot/dns-cloudflare:latest
                volumes:
                        - ./volumes/certbot/www/:/var/www/certbot/:rw
                        - ./volumes/certbot/conf/:/etc/letsencrypt/:rw
                        - /root/cred.ini:/root/cred.ini
```

### Reverse Proxy

Setup Nginx from my docker compose file to run on a bridged network. Had to deconflict port 80 from pihole on the same host.

Default config to accept the challenge for the certbot.

```
server {
    listen 80;
    listen [::]:80;
    server_name dnd.spaceforce.xxx;
    
    location ~ /.well-known/acme-challenge {
        allow all;
        root /var/www/certbot;
    }
}
```
All other configuration is stored in each file by domain and proxy in ./volumes/nginx/conf

automation.conf  base.conf  challenge.conf  dnd.conf  nvr.conf  tv.conf
