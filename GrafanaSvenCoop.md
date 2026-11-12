# Sven-Coop Server Grafana Monitor
Die Fortführung der Grafana Guides, dieses Mal verbinden wir das Hitspiel Sven-Coop mit InfluxDB und Grafana über A2S und Python

### Im Grafana Container:
```
apt install python3-pip -y
```
```
pip3 install python-a2s --break-system-packages
```

### Bucket und Token von InfluxDB erstellen
In InfluxDB: Load Data -> Buckets -> Create Bucket

| Feld           | Wert           |
| -------------- |----------------|
| Name           | svencoop       |  

In InfluxDB: Load Data -> API Tokens -> Generate API Token -> Custom API Token

| Feld           | Wert           |
| -------------- |----------------|
| Name           | svencoop       |
| Bucket         | temperatures   |
| Rechte         | Read, Write    |

### Das Python Script anlegen:
```
nano /opt/sc_a2s_poller.py
```
Bitte das Token und die Sven Coop Server IP eintragen und gegebenenfalls die Organisation überprüfen
```
import time
import urllib.request
import a2s

# --- KONFIGURATION ---
SERVER_ADDRESS = ("SVENCOOP SERVER IP", 27015)  # IP und Port deines Sven Co-op LXC!
INFLUX_URL = "http://localhost:8086/api/v2/write?org=homelab&bucket=svencoop&precision=s"
INFLUX_TOKEN = "DEIN_API_TOKEN"
POLL_INTERVAL = 15  # Abfrage-Intervall in Sekunden
# ---------------------

def escape_tag(val):
    if not val:
        return "unknown"
    return str(val).replace(" ", "\\ ").replace(",", "\\,").replace("=", "\\=")

def send_to_influx(lines):
    if not lines:
        return
    payload = "\n".join(lines).encode("utf-8")
    req = urllib.request.Request(INFLUX_URL, data=payload, method="POST")
    req.add_header("Authorization", f"Token {INFLUX_TOKEN}")
    req.add_header("Content-Type", "text/plain; charset=utf-8")
    try:
        urllib.request.urlopen(req, timeout=3)
    except Exception as e:
        print(f"InfluxDB Fehler: {e}")

while True:
    lines = []
    try:
        # 1. Server-Infos abrufen (A2S_INFO)
        info = a2s.info(SERVER_ADDRESS, timeout=3.0)
        map_name = escape_tag(info.map_name)
        server_name = escape_tag(info.server_name)
        
        lines.append(
            f"sc_server,server={server_name},map={map_name} "
            f"players={info.player_count}i,max_players={info.max_players}i,bots={info.bot_count}i,online=1i"
        )

        # 2. Aktive Spieler abrufen (A2S_PLAYER)
        players = a2s.players(SERVER_ADDRESS, timeout=3.0)
        for p in players:
            if p.name:  # Leere Slots (Spieler gerade am Verbinden) ignorieren
                p_name = escape_tag(p.name)
                lines.append(
                    f"sc_players,player={p_name},map={map_name} "
                    f"score={p.score}i,duration={int(p.duration)}i"
                )

    except Exception as e:
        # Wenn der Server offline ist oder nicht antwortet, online=0 loggen
        lines.append("sc_server online=0i,players=0i")

    send_to_influx(lines)
    time.sleep(POLL_INTERVAL)
```
### Als Hintergrund Dienst einrichten:
```
nano /etc/systemd/system/sc-a2s.service
```
```
[Unit]
Description=Sven Co-op A2S to InfluxDB2 Poller
After=network.target influxdb.service

[Service]
ExecStart=/usr/bin/python3 /opt/sc_a2s_poller.py
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```
### Dienst starten:
```
systemctl daemon-reload
```
```
systemctl start sc-a2s
```
### Query Top Karten nach Spielzeit:
```
from(bucket: "svencoop")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r._measurement == "sc_server" and r._field == "players" and r._value > 0)
  |> group(columns: ["map"])
  |> count()
  |> map(fn: (r) => ({ r with Minuten: float(v: r._value) * 15.0 / 60.0 }))
  |> keep(columns: ["map", "Minuten"])
  |> group()
  |> sort(columns: ["Minuten"], desc: true)
```
### Query Top Spieler nach Punkten:
```
from(bucket: "svencoop")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r._measurement == "sc_players" and r._field == "score")
  |> group(columns: ["player"])
  |> increase()
  |> last()
  |> keep(columns: ["player", "_value"])
  |> rename(columns: {_value: "Gesamtpunkte"})
  |> group()
  |> sort(columns: ["Gesamtpunkte"], desc: true)
```
