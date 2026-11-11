# Temperatur vom Proxmox Host zu Grafana
Ein kurzer Guide wie man die CPU Temperaturen vom Proxmox Host zu InfluxDB und Grafana durchreicht

### Sensoren testen (Im Container):
```
apt install lm-sensors gnupg -y
```
```
sensors
```
Temperaturen sollten angezeigt werden

### Telegraf installieren:
```
curl -fsSL https://repos.influxdata.com/influxdata-archive.key | gpg --dearmor -o /etc/apt/keyrings/influxdata-archive.gpg
```
```
echo 'deb [signed-by=/etc/apt/keyrings/influxdata-archive.gpg] https://repos.influxdata.com/ubuntu stable main' > /etc/apt/sources.list.d/influxdata.list
```
```
apt update && apt install telegraf -y
```
Wenn die Meldung kommt wegen Modified configuration file einfach auf keep local version

### Bucket und Token von InfluxDB erstellen
In InfluxDB: Load Data -> Buckets -> Create Bucket

| Feld           | Wert           |
| -------------- |----------------|
| Name           | temperatures   |  

In InfluxDB: Load Data -> API Tokens -> Generate API Token -> Custom API Token

| Feld           | Wert           |
| -------------- |----------------|
| Name           | temp-write     |
| Bucket         | temperatures   |
| Rechte         | Read, Write    |

### Konfigurationsdatei anlegen und Dienst starten:
```
nano /etc/telegraf/telegraf.d/temperature.conf
```
```
[agent]
  interval = "5s"
  flush_interval = "5s"

[[inputs.temp]]

[[outputs.influxdb_v2]]
  urls = ["http://IP-Adresse:8086"]
  token = "temp-write Token hier einsetzen"
  organization = "homelab"
  bucket = "temperatures"
```
```
systemctl start telegraf
```
### Temperatur Query erstellen:
```
from(bucket: "temperatures")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "temp")
  |> filter(fn: (r) =>
    r["sensor"] == "coretemp_core_0" or
    r["sensor"] == "coretemp_core_1" or
    r["sensor"] == "coretemp_core_2" or
    r["sensor"] == "coretemp_core_3" or
    r["sensor"] == "coretemp_package_id_0"
  )
  |> map(fn: (r) => ({
    r with
    sensor_name:
      if r.sensor == "coretemp_core_0" then "CPU Kern 0"
      else if r.sensor == "coretemp_core_1" then "CPU Kern 1"
      else if r.sensor == "coretemp_core_2" then "CPU Kern 2"
      else if r.sensor == "coretemp_core_3" then "CPU Kern 3"
      else if r.sensor == "coretemp_package_id_0" then "CPU Gesamt"
      else r.sensor
  }))
  |> group(columns: ["sensor_name"])
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "Temperaturen")
```
