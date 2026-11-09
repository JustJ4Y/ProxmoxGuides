# Dateifreigabe mit Samba
Ein kleiner Guide wie man eine Netzwerkfreigabe mit Samba erstellt

### Container erstellen:
```
LXC - Ubuntu 24.04 (Privileged)
```
### Auf Container:
```
apt update && apt upgrade -y
```
```
apt install samba -y
```
```
mkdir /sambashare
```
### Neuen Nutzer für Samba anlegen:
```
useradd Samba
```
```
passwd Samba
```
```
smbpasswd -a Samba
```
```
chown -R Samba /sambashare
```
### Samba Konfiguration bearbeiten:
```
sudo nano /etc/samba/smb.conf
```
### Unten hinzufügen:
```
[sambashare]  
(>) path = /sambashare  
(>) read only = no  
(>) browsable = yes
```
### Danach:
```
systemctl restart smb
```
### Wenns noch nicht funktioniert, Berechtigung und Firewall checken:
```
chmod 755 sambashare
```
```
sudo ufw allow samba
```
