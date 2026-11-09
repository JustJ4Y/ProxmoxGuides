# Proxmox Monitoring mit Grafana
Guide zum erstellen eines LXC Containers mit Docker auf dem InfluxDB 2 und Grafana

### Container erstellen:
```
LXC - Ubuntu 24.04 (Unprivileged)
```
### Auf Container:
```
apt update && apt upgrade -y
```
### Docker installieren:
Falls es Probleme bei diesem Schritt gibt, schaut Mal in den ```DockerApparmor.ad``` Guide
```
apt install -y ca-certificates curl gnupg lsb-release
```
```
install -m 0755 -d /etc/apt/keyrings
```
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```
```
chmod a+r /etc/apt/keyrings/docker.asc
```
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu apt update
```
```
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
```
systemctl enable --now docker
```
### Docker testen und Docker Compose Datei erstellen:
```
docker --version
```
```
mkdir -p ~/monitoring
```
```
cd ~/monitoring
```
```
nano docker-compose.yml
```
### In Datei einfügen:
```
services:  
 influxdb:  
 image: influxdb:2  
 container_name: influxdb  
 restart: unless-stopped  
 ports:  
 - "8086:8086"  
 volumes:  
 - influxdb-data:/var/lib/influxdb2  
 - influxdb-config:/etc/influxdb2  
 grafana:  
 image: grafana/grafana:latest  
 container_name: grafana  
 restart: unless-stopped  
 ports:  
 - "3000:3000"  
 volumes:  
 - grafana-data:/var/lib/grafana  
 depends_on:  
 - influxdb  
volumes:  
 influxdb-data:  
 influxdb-config:  
 grafana-data:  
```
### Docker starten und Status checken:
```
docker compose up -d
```
```
docker ps
```
### InfluxDB einrichten:
InfluxDB im Browser öffnen unter Port 8086  
| Feld           | Wert           |
| -------------- |----------------|
| Username       | admin          |
| Organization   | homelab        |
| Bucket         | proxmox        |
Den initialen Token speichern. Fuer Proxmox spaeter einen eigenen Write-Token erstellen.

### Grafana Data Source einrichten:
Grafana im Browser öffnen unter Port 3000  
Standardloging ist admin/admin
| Feld           | Wert                         |
| -------------- |------------------------------|
| Query language | Flux                         |
| URÖ            | http://influxdb:8086         |
| Organization   | homelab                      |
| Default Bucket | proxmox                      |
| Token          | Grafana-Token hier einsetzen |
Erfolgreicher Test: Data source is working. 3 buckets found


