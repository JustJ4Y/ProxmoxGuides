### VPN Dienst mit Wireguard
Falls ihr Dienste von außen erreichen wollte, ohne sie dem Internet zu öffnen, benutzt ihr am besten einen VPN.

### Container erstellen:
```
LXC - Ubuntu 24.04 (Privileged)
```
### Auf Container:
```
apt update && apt upgrade -y
```
```
apt install wireguard iptables -y
```
### Konfig erstellen lassen:
```
www.wireguardconfig.com
```
(Auf richtigen Endpoint und Port achten)
### Wireguard Konfig bearbeiten:
```
nano /etc/wireguard/wg0.conf
```
### Server Config einfügen und das hinzufügen über ```PostUP = iptables```:
```
PostUp = sysctl -w net.ipv4.ip_forward=1
```
### Testen:
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
Wenns nicht läuft, eventuell KI fragen, kann viele Gründe haben.
