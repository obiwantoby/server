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

Documentation is at https://ubuntu.com/server/docs/backups-shell-scripts

```

ip:/volume1/Backups /mnt/backup nfs defaults 0 0
```
