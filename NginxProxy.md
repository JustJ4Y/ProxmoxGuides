# Nginx Proxy Manager installieren
Ein recht einfacher Guide zum installieren vom Nginx Proxy Manager mit Docker

### Container erstellen:
```
LXC - Ubuntu 24.04 (Unprivileged)
```
### Auf Container:
```
apt update && apt upgrade -y
```
```
apt install curl
```
### Docker installieren:
Falls es Probleme bei diesem Schritt gibt, schaut Mal in den ```DockerApparmor.ad``` Guide
```
curl -sSL https://get.docker.com | sh
```
```
apt install docker-compose
```
### Docker Compose Datei erstellen:
```
nano docker-compose.yml
```
### In Konfiguration einfügen:
```
services:  
  app:  
    image: 'docker.io/jc21/nginx-proxy-manager:latest'  
    restart: unless-stopped  
    ports:  
      - '80:80'  
      - '81:81'  
      - '443:443'  
    volumes:  
      - ./data:/data  
      - ./letsencrypt:/etc/letsencrypt  
```
### Docker starten:
```
docker-compose up -d
```
Das Webinterface ist jetzt zu finden unter Port 81
