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
```
apt install curl -y
```
### Docker installieren:
Falls es später Probleme mit Docker gibt, schaut Mal in den ```DockerApparmor.ad``` Guide
```
curl -sSL https://get.docker.com | sh
```
### Docker Compose Datei erstellen:
```
nano docker-compose.yml
```
### In Konfiguration einfügen:
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
### Docker starten:
```
docker compose up -d
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
In Grafana: Connections -> Data sources -> Add data source -> InfluxDB

| Feld           | Wert                         |
| -------------- |------------------------------|
| Query language | Flux                         |
| URL            | http://influxdb:8086         |
| Organization   | homelab                      |
| Token          | Grafana-Token hier einsetzen | 
| Default Bucket | proxmox                      |


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
In Proxmox: Datacenter -> Metric Server -> Add -> InfluxDB

| Feld           | Wert                       |
| -------------- |----------------------------|
| Name           | influxdb                   |
| Server         | IP-Adresse vom LXC         |
| Port           | 8086                       |
| Protokoll      | http                       |
| Organization   | homelab                    |
| Bucket         | proxmox                    |
| Token          | Write-Token hier einsetzen |  

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
