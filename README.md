# Home Server scripts
Home Server Files

## General Server Setup

**Debian 11**
**Global Unit Files** for systemctl located in /etc/systemd/system/
**User Unit Files** for systemctl --user located in

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
