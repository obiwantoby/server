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


## 🚀 Proxmox Network Tuning – Final Applied Settings

These settings were used to optimize WAN upload performance and reduce latency/bufferbloat, while keeping LAN throughput strong.  
They’re safe to reapply after reboots via a persistent script/systemd service.

### 1. TCP & Queue Discipline
```bash
# Load and set BBR congestion control
modprobe tcp_bbr
sysctl -w net.ipv4.tcp_congestion_control=bbr

# Apply fair queuing (fq) to NIC
tc qdisc add dev enp5s0f0np0 root fq
```

### 2. NIC Offload Adjustments *(optional if router handles shaping)*
```bash
# Reduce burstiness by disabling large segment offloads
ethtool -K enp5s0f0np0 gso off tso off lro off

# Keep GRO enabled for receive efficiency
ethtool -K enp5s0f0np0 gro on
```

### 3. Queue & Interrupt Tuning
```bash
# Limit TX queue length
ip link set dev enp5s0f0np0 txqueuelen 1000

# Enable adaptive interrupt coalescing
ethtool -C enp5s0f0np0 adaptive-rx on adaptive-tx on rx-usecs 50 tx-usecs 50
```

### 4. Persistence
- `/etc/sysctl.d/99-network-bbr.conf`:
  ```
  net.ipv4.tcp_congestion_control = bbr
  ```
- Systemd service (`/etc/systemd/system/network-tune.service`) to reapply `fq` at boot:
  ```ini
  [Unit]
  Description=Apply fq qdisc on enp5s0f0np0 at boot
  After=network.target

  [Service]
  Type=oneshot
  ExecStart=/usr/sbin/tc qdisc add dev enp5s0f0np0 root fq
  RemainAfterExit=yes

  [Install]
  WantedBy=multi-user.target
  ```

Enable with:
```bash
systemctl daemon-reexec
systemctl enable network-tune.service
```

### 5. OPNsense Router Shaping
- Enabled **FQ-CoDel** on WAN.
- Recommended: Also enable **FQ-CoDel or Cake** on LAN to pace egress toward WAN and enforce fairness between LAN clients.

---

**Note:**  
- Host‑side tuning was key for diagnostics and per‑node optimization.  
- Router‑side shaping (FQ-CoDel/Cake) ensures multi‑client fairness and consistency.  
- Offload settings may be re‑enabled for max efficiency if central shaping is active and CPU load is a concern.

