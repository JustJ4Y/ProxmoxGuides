# HP C4400 als Netzwerkdrucker/Scanner
Einen alten Drucker/Scanner Netzwerkfähig machen mit CUPS und ScanservJS  

### Container erstellen:
```
LXC - Ubuntu 24.04 (Privileged)
```
### Auf Proxmox Host:
```
lsusb
```  
Drucker sollte hier gelistet werden. 
### Dann Container bearbeiten:
```
nano /etc/pve/lxc/containernummer.conf
```
### Zeilen hinzufügen:
```
features: nesting=1,keyctl=1  
lxc.cgroup2.devices.allow: c 189:* rwm  
lxc.mount.entry: /dev/bus/usb dev/bus/usb none bind,optional,create=dir
```
Alle USB Geräte werden durchgeschleust.
### Weiter auf Container:
```
apt update && apt upgrade -y
```
```
apt install -y cups cups-client printer-driver-hpcups hplip hplip-data sane sane-utils \  
libsane1 avahi-daemon usbutils curl
```
### Checken ob der Drucker korrekt durchgereicht wird:
```
lsusb
```
```
hp-info
```
```
hp-check
```
### Installieren (Fehlermeldung kann man meist ignorieren):  
```
hp-setup -i
```
```
lpinfo -v  
```
```
lpstat -p  
```

### In Cups Config checken:
```
nano /etc/cups/cupsd.conf  
```
### Gegebenfalls bearbeiten:
```
Listen 0.0.0.0:631  
Browsing On  
BrowseLocalProtocols dnssd  
DefaultAuthType Basic  
WebInterface Yes
```
### Bei den <Location>-Blöcken nach geschmack anpassen (``` Require user @SYSTEM ```ersetzen):  
```
Order allow, deny  
Allow @LOCAL
```
### Damach neustarten:
```
systemctl restart cups
```
Drucker sollte laufen, Webinterface unter Port 631.
### Scanner testen:
```
scanimage -L
```
```
scanimage > test.pnm
```
### Installieren von Scanservjs:
```
wget https://github.com/sbs20/scanservjs/releases/download/v3.2.0/scanservjs_3.2.0-1_all.deb
```
```
apt install ./scanservjs_3.2.0-1_all.deb
```
```
systemctl enable scanservjs
```
Webinterface für Scanner erreichbar unter Port 8080.  
### cups, avahi-daemon und scanservjs in Autostart packen: 
```
systemctl enable <Servicename>
```
### Falls PDF Scans nicht funktionieren:
```
nano /etc/ImageMagick-6/policy.xml
```

### Diese Zeile auskommentieren mit ``` <!-- -->```:
```
<policy domain="path" rights="none" pattern="@*"/>
```
```
systemctl restart scanservjs
```
Fertig, es sollte alles laufen, viel Spaß
