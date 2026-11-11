# Sven Coop Server
Guide zum erstellen eines Sven Coop Server für maximalen Spielspaß, andere Goldsource Spiele funktionieren ähnlich

### Container erstellen:
```
LXC - Ubuntu 24.04 (Unprivileged)
```
### Auf Container:
```
apt update && apt upgrade -y
```
### SteamCMD installieren:
```
dpkg --add-architecture i386
```
```
apt update && apt install zlib1g:i386 steamcmd -y
```
### SteamCMD Nutzer anlegen:
```
useradd -m sven
```
```
passwd sven
```
```
su sven
```
### Sven Coop installieren:
```
steamcmd +force_install_dir ~ +login anonymous +app_update 276060 validate +quit
```
Portforwarding im Router UDP:27015
### Server starten:
```
cd ~ && ./svends_run -console -port 27015 +maxplayers 10 +log on +map abandoned -binary ./svends_i686
```
