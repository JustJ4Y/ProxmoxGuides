# Wichtige Befehle
Ein paar Befehle von den Linuxbasics bis zu Proxmox und Docker  

### Navigation, Datei und Ordner erstellen

| Befehl         | Funktion                                                  |
| -------------- |-----------------------------------------------------------|
| cd             | Verzeichnis wechseln                                      |
| ls             | Ordnerinhalt anzeigen (-l für Berechtigungen)             |
| cat            | Dateiinhalt anzeigen                                      |
| rm             | Datei löschen (-r Rekursiv für Ordner)                    |
| mv             | Datei verschieben                                         |
| cp             | Datei kopieren                                            |
| nano           | Texteditor (wenn nicht vorhanden, erstellt Datei)         |
| mkdir          | Ordner erstellen                                          |  

### Management, Installation und Einblicke

| Befehl         | Funktion                                                  |
| -------------- |-----------------------------------------------------------|
| sudo           | Ausführen mit Adminrechten                                |
| find           | Nach Datein und Orndern suchen (-name)                    |
| grep           | Durchsucht den Inhalt von Datein                          |
| man            | Erklärung zu den Befehlen anzeigen                        |
| apt            | Paketverwaltung (update, upgrade, install)                |
| ip a           | IP-Adressen anzeigen                                      |
| top            | Systemauslastung                                          |
| ps aux         | Laufende Prozesse anzeigen                                |
| lsblk          | Festplattenpartitionen anzeigen                           |
| reboot         | System Neustarten                                         |


### Dateiberechtigungen und Benutzerverwaltung

| Befehl         | Funktion                                                  |
| -------------- |-----------------------------------------------------------|
| chmod          | Berechtigungen ändern (+rwx oder durch Oktalzahl,zB. 755) |
| chown          | Datei anderem Nutzer oder Gruppe zuweisen                 |
| useradd        | User erstellen (-m für Home, -s /bin/bash für die Shell)  |
| passwd         | Passwort ändern                                           |
| userdel        | Benutzer löschen (am besten alles mit -r)                 |
| users          | Auflisten aller Benutzer                                  |
| su             | Benutzer wechseln                                         |
| id             | Benutzerinformationen anzeigen                            |

### Proxmox, Dienste, LXC und Docker

| Befehl         | Funktion                                                  |
| -------------- |-----------------------------------------------------------|
| systemctl      | Systemdienste (start, stop, status, restart)              |
| qm             | Virtuelle Maschinen (list, start, config, destroy)        |
| pct            | LXC-Container Funktionen (list, start, stop, destroy)     |
| pct enter      | Öffnet eine Shell im LXC Container                        |
| docker ps      | Listet laufende Docker-Container auf                      |
| docker compose | Docker Compose (up -d zum Starten, pull zum Updaten)      |

### Wichtige Verzeichnisse und Konfigurationsdateien

| Verzeichnis    | Funktion                                                  |
| -------------- |-----------------------------------------------------------|
| /home          | Persönliche Nutzerverzeichnisse                           |
| /var/log       | Logdateien                                                |
| /etc           | Konfigurationsdatein (shadow, network, netplan, ssh)      |
| /mnt           | Einhängepunkt für Laufwerke und Wecheselmedien            |
| /etc/pve/lxc   | Konfigurationsdatein der LXC-Container                    |




