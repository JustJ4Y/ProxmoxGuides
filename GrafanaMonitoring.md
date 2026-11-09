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

Den initialen Token speichern. Für Proxmox später einen eigenen Write-Token erstellen.

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

Erfolgreicher Test: ```Data source is working. 3 buckets found```

### Proxmox Write-Token in InfluxDB erstellen:
In InfluxDB: Load Data -> API Tokens -> Generate API Token -> Custom API Token

| Feld           | Wert           |
| -------------- |----------------|
| Name           | proxmox-write  |
| Bucket         | proxmox        |
| Rechte         | Read, Write    |  

Den Token komplett kopieren und speichern.

### Proxmox Metric Server einrichten:

| Feld           | Wert                       |
| -------------- |----------------------------|
| Name           | influxdb                   |
| Port           | 8086                       |
| Organization   | homelab                    |
| Bucket         | proxmox                    |
| Token          | Write-Token hier einsetzen |  

### Auf Proxmox Host testen:
```
cat /etc/pve/status.cfg
```
```
journalctl -u pvestatd -f
```
### CPU Query in Grafana erstellen:
```
from(bucket: "proxmox")
 |> range(start: -1h)
 |> filter(fn: (r) => r["vmid"] =~ /^(100|101|102|103|105|106|107|108|109)$/)
 |> filter(fn: (r) => r["_field"] == "cpu")
 |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
 |> map(fn: (r) => ({
 r with
 _value: r._value * 100.0,
 _field:
 if r.vmid == "100" then "Server 0"
 else if r.vmid == "101" then "Server 1"
 else if r.vmid == "102" then "Server 2"
 else if r.vmid == "103" then "Server 3"
 else if r.vmid == "105" then "Server 4"
 else if r.vmid == "106" then "Server 5"
 else if r.vmid == "107" then "Server 6"
 else if r.vmid == "108" then "Server 7"
 else if r.vmid == "109" then "Server 8"
 else "Unbekannt"
 }))
 |> keep(columns: ["_time", "_value", "_field"])
 |> group(columns: ["_field"])
 |> yield(name: "CPU Dienste")
```
### RAM Query in Grafana erstellen:
```
from(bucket: "proxmox")
 |> range(start: -1h)
 |> filter(fn: (r) => r["vmid"] =~ /^(100|101|102|103|105|106|107|108|109)$/)
 |> filter(fn: (r) => r["_field"] == "mem")
 |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
 |> map(fn: (r) => ({
 r with
 _field:
 if r.vmid == "100" then "Server 0"
 else if r.vmid == "101" then "Server 1"
 else if r.vmid == "102" then "Server 2"
 else if r.vmid == "103" then "Server 3"
 else if r.vmid == "105" then "Server 4"
 else if r.vmid == "106" then "Server 5"
 else if r.vmid == "107" then "Server 6"
 else if r.vmid == "108" then "Server 7"
 else if r.vmid == "109" then "Server 8"
 else "Unbekannt"
 }))
 |> keep(columns: ["_time", "_value", "_field"])
 |> group(columns: ["_field"])
 |> yield(name: "RAM Dienste")
```
