# Proxmox Monitoring mit Grafana
Guide zum erstellen eines Grafana LXC Containers mit Hilfe von InfluxDB 2 und Docker

## Installation

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

Jetzt sollte du soweit sein, dass du dein erstes Dashboard erstellen und dort die ersten Queries ausprobieren kannst.

## Query

### LXC CPU Auslastung Grafana:
Display name: ${__field.labels.host}  
Unit: Percent (0.0-1.0)
```
  from(bucket: "proxmox")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "system")
  |> filter(fn: (r) => r["_field"] == "cpu")
  |> filter(fn: (r) => r["object"] == "lxc")
  |> filter(fn: (r) => r["_value"] > 0)
  |> filter(fn: (r) => r["nodename"] == "proxmox")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
//  |> yield(name: "mean
```
### LXC RAM Auslastung Grafana:
Display name: ${__field.labels.host}  
Unit: bytes(SI)
```
  from(bucket: "proxmox")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "system")
  |> filter(fn: (r) => r["_field"] == "mem")
  |> filter(fn: (r) => r["object"] == "lxc")
  |> filter(fn: (r) => r["_value"] > 0)
  |> filter(fn: (r) => r["nodename"] == "proxmox")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
//  |> yield(name: "mean")
```
### LXC Netzwerk Auslastung Grafana:
Display name: ${__field.labels.host}  
Unit: bytes(SI)
```
from(bucket: "proxmox")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "system")
  |> filter(fn: (r) => r["_field"] == "netin") or r["_field"] == "netout")
  |> filter(fn: (r) => r["object"] == "lxc")
  |> filter(fn: (r) => r["_value"] > 0)
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> derivative(unit: 1s, nonNegative: true)
//  |> yield(name: "mean")
```
### CPU Host Auslastung in Prozent:
Unit: Percent (0-100)
```
from(bucket: "proxmox")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) =>
    r._measurement == "cpustat" and
    r._field == "cpu" and
    r.host == "proxmox"
  )
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> map(fn: (r) => ({
    r with
    _value: r._value * 100.0,
    _field: "CPU Auslastung"
  }))
  |> keep(columns: ["_time", "_value", "_field"])
  |> yield(name: "CPU")
  ```
  ### RAM Host Auslastung in Prozent:  
  Unit: Percent (0-100)  
  ```
  from(bucket: "proxmox")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) =>
    r._measurement == "memory" and
    (r._field == "memused" or r._field == "memtotal") and
    r.host == "proxmox"
  )
  |> aggregateWindow(every: v.windowPeriod, fn: last, createEmpty: false)
  |> pivot(
    rowKey: ["_time"],
    columnKey: ["_field"],
    valueColumn: "_value"
  )
  |> map(fn: (r) => ({
    r with _value: float(v: r.memused) / float(v: r.memtotal) * 100.0,
    _field: "RAM Auslastung"
  }))
  |> keep(columns: ["_time", "_value", "_field"])
  |> yield(name: "RAM")
```
