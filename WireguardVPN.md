# VPN Dienst mit Wireguard
Falls ihr Dienste von außen erreichen wollte, ohne sie dem Internet zu öffnen, benutzt ihr am besten einen VPN.

### Container erstellen:
```
LXC - Ubuntu 24.04 (Unprivileged)
```
### Auf Container:
```
apt update && apt upgrade -y
```
```
apt install wireguard iptables -y
```
### Konfiguration erstellen lassen:  

https://www.wireguardconfig.com

(Auf richtigen Endpoint achten und eventuell gewünschten Port setzen)
### Wireguard Konfiguration bearbeiten:
```
nano /etc/wireguard/wg0.conf
```
### Server Konfiguration einfügen und das hinzufügen über ```PostUP = iptables```:
```
PostUp = sysctl -w net.ipv4.ip_forward=1
```
### Port im Router forwarden und Dienst starten:
```
wg-quick up wg0
```
Danach versuchen mit Client zu verbinden
### Status checken:
```
wg
```
### In Autostart packen:
```
systemctl enable wg-quick@wg0
```
Wenns nicht läuft oder zwischendurch nicht weitergeht, eventuell KI fragen, kann viele Gründe haben. Aber schaut euch gerne die Firewallregeln an und ob bei Endpoint und Port alles richtig gesetzt wurde. Die ufw von Ubuntu kann auch manchmal dazwischen funken. Dann probiert:
```
ufw allow Portnummer
```
