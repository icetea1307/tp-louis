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
