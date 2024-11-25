### fait par aguer hugo 


### Étape 1 : Analyse et nettoyage du serveur

```
1.Lister les tâches cron pour détecter des backdoors :
```
```
[root@localhost ~]# sudo cat /var/log/cron
```

### 2.Identifier et supprimer les fichiers cachés :
```
[root@localhost ~]# ls -alh /tmp
```
```
-rwxrwxrwx.  1 attacker attacker   18 Nov 24 18:24 .hidden_file
-rwxrwxrwx.  1 attacker attacker   17 Nov 24 18:11 .hidden_script
-rwxr-xr-x.  1 attacker attacker   23 Nov 24 18:11 malicious.sh
```
```
[root@localhost ~]# ls -alh /var/tmp
```
```
-rwxrwxrwx.  1 attacker attacker    7 Nov 24 20:10 .nop
```
```
[root@localhost ~]# ls -alh /home
```
```
drwx------.  2 attacker attacker 4.0K Nov 24 20:09 attacker
```
```
[root@localhost ~]# sudo rm -r /home/attacker
[root@localhost ~]# sudo rm -r /tmp/.hidden_file
[root@localhost ~]# sudo rm -r /tmp/.hidden_script
[root@localhost ~]# sudo rm -r /tmp/malicious.sh
[root@localhost ~]# sudo rm -r /var/tmp/.nop
```
```
[root@localhost ~]# sudo ss -tunap
Netid    State     Recv-Q    Send-Q              Local Address:Port         Peer Address:Port     Process
udp      ESTAB     0         0            192.168.245.3%enp0s8:68          192.168.245.2:67        users:(("NetworkManager",pid=872,fd=32))
udp      ESTAB     0         0                10.0.2.15%enp0s3:68               10.0.2.2:67        users:(("NetworkManager",pid=872,fd=29))
udp      UNCONN    0         0                       127.0.0.1:323               0.0.0.0:*         users:(("chronyd",pid=868,fd=5))
udp      UNCONN    0         0                           [::1]:323                  [::]:*         users:(("chronyd",pid=868,fd=6))
tcp      LISTEN    0         128                       0.0.0.0:22                0.0.0.0:*         users:(("sshd",pid=921,fd=3))
tcp      ESTAB     0         0                   192.168.245.3:22          192.168.245.1:26879     users:(("sshd",pid=1593,fd=4),("sshd",pid=1589,fd=4))
tcp      LISTEN    0         128                          [::]:22                   [::]:*         users:(("sshd",pid=921,fd=4))
```

### Étape 2 : Configuration avancée de LVM

### 1.Créer un snapshot de sécurité pour /mnt/secure_data :
```
[root@localhost ~]# sudo lvcreate --size 500.00mo --snapshot --name snap /dev/vg_secure/secure_data
  Logical volume "snap" created.
```
### 2. Tester la restauration du snapshot :
```
sudo lvcreate --size 500.00mo --snapshot --name snap /dev/vg_secure/secure_data
```
```
[root@localhost ~]# ls /mnt/secure_data/
[root@localhost ~]# sudo rm /mnt/secure_data/sensitive2.txt
[root@localhost ~]# sudo mkdir /mnt/snapshot
[root@localhost ~]# sudo mount /dev/vg_secure/snap /mnt/snapshot
[root@localhost ~]# cp /mnt/snapshot/sensitive2.txt /mnt/secure_data/
[root@localhost ~]# sudo umount /mnt/snapshot/
[root@localhost ~]# sudo vgdisplay
```


### 3 .Optimiser l’espace disque :
```
[root@localhost ~]# lvextend --size +10.00m /dev/vg_secure/secure_data
  Rounding size to boundary between physical extents: 12.00 MiB.
  Size of logical volume vg_secure/secure_data changed from 500.00 MiB (125 extents) to 512.00 MiB (128 extents).
  Logical volume vg_secure/secure_data successfully resized.
```
### Etape 3.Automatisation avec un script de sauvegarde
### 1.Créer un script secure_backup.sh :
```
[root@localhost ~]# sudo nano /usr/local/bin/secure_backup.sh
[root@localhost ~]# sudo chmod +x /usr/local/secure_backup.sh

```
```
2.
#!/bin/bash

Variables
SOURCEDIR="/mnt/secure_data"
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d%H%M)
BACKUPFILE="$BACKUP_DIR/secure_data$DATE.tar.gz"
MAX_BACKUPS=7
LOG_FILE="/var/log/backup.log"

Création de l'archive en excluant les fichiers temporaires et cachés
tar --exclude='.tmp' --exclude='.log' --exclude='.*' -czf "$BACKUP_FILE" -C "$SOURCE_DIR" . >> "$LOG_FILE" 2>&1

Fonction de rotation des sauvegardes
rotate_backups() {
    cd "$BACKUP_DIR" || exit
    if [ $(ls -1 | wc -l) -gt $MAX_BACKUPS ]; then
        ls -tp | grep -v '/$' | tail -n +$((MAX_BACKUPS + 1)) | xargs -I {} rm -- {}
    fi
}

Appel de la fonction de rotation
rotate_backups

echo "Sauvegarde créée : $BACKUP_FILE" >> "$LOG_FILE"
```
```
3.Testez le script :
```
```
[root@localhost ~]# sudo /usr/local/bin/secure_backup.sh
[root@localhost ~]# ls /backup
secure_data_2024111227.tar.gz     secure_data_20241125_1230.tar.gz  secure_data_20241125_1232.tar.gz  secure_data_20241125_1234.tar.gz
secure_data_20241125_1229.tar.gz  secure_data_20241125_1231.tar.gz  secure_data_20241125_1233.tar.gz
```
### 4.Automatisez avec une tâche cron :
```
[root@localhost ~]# sudo crontab -e
no crontab for root - using an empty one
crontab: installing new crontab
1 3 * * * /usr/local/bin/secure_backup.sh
```
### Étape 4 : Surveillance avancée avec auditd

### 1.Configurer auditd pour surveiller /etc :
```
[root@localhost ~]# sudo auditctl -w /etc -p wa -k etc_changes
Old style watch rules are slower
```
### 2.Tester la surveillance
```
[root@localhost ~]# sudo auditctl -l
-w /etc -p wa -k etc_changes
```