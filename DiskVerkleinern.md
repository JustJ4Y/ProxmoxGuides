# Disk eines LXC Containers verkleinern (ext4)
Falls ihr Mal die Disk eines LXCs verkleinern wollt oder müsst, hier am Beispiel von Container 100 auf 4GB runter. Auf jeden Fall auf eigenes Risiko, am besten vorher Backup machen. 

### Auf Proxmox Host:  
```
pct stop 100
```
### Dateisystem prüfen und verkleinern:
```
e2fsck -f /dev/pve/vm-100-disk-0
```
```
resize2fs /dev/pve/vm-100-disk-0 3800M
```
```
lvreduce -L 4G /dev/pve/vm-100-disk-0
```
### In der Konfiguration auch bearbeiten:
```
nano /etc/pve/lxc/100.conf
```
No Risk, no Fun, jetzt kann man ihn wieder starten und gucken was passiert
```
pct start 100
```
