# TP2 : Environnement virtuel

# 0. Prérequis

# I. Topologie réseau

## Compte-rendu

☀️ Sur **`node1.lan1.tp1`**

- afficher ses cartes réseau
```
[quentin@node1-lan1-tp1 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:05:9e:77 brd ff:ff:ff:ff:ff:ff
    inet 10.1.1.11/24 brd 10.1.1.255 scope global noprefixroute enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe05:9e77/64 scope link
       valid_lft forever preferred_lft forever
```

- afficher sa table de routage
```
[quentin@node1-lan1-tp1 ~]$ ip r s
10.1.1.0/24 dev enp0s3 proto kernel scope link src 10.1.1.11 metric 100
10.1.2.0/24 via 10.1.1.254 dev enp0s3 proto static metric 100
```

- prouvez qu'il peut joindre `node2.lan2.tp2`
```
[quentin@node1-lan1-tp1 ~]$ ping 10.1.2.12
PING 10.1.2.12 (10.1.2.12) 56(84) bytes of data.
64 bytes from 10.1.2.12: icmp_seq=1 ttl=63 time=0.503 ms
64 bytes from 10.1.2.12: icmp_seq=2 ttl=63 time=2.19 ms
^C
--- 10.1.2.12 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1025ms
rtt min/avg/max/mdev = 0.503/1.346/2.190/0.843 ms
```

- prouvez avec un `traceroute` que le paquet passe bien par `router.tp1`
```
[quentin@node1-lan1-tp1 ~]$ traceroute 10.1.2.12
traceroute to 10.1.2.12 (10.1.2.12), 30 hops max, 60 byte packets
 1  10.1.1.254 (10.1.1.254)  0.417 ms  0.502 ms  0.499 ms
 2  10.1.2.12 (10.1.2.12)  1.564 ms !X  1.561 ms !X  1.555 ms !X
```


# II. Interlude accès internet

☀️ **Sur `router.tp1`**

- prouvez que vous avez un accès internet (ping d'une IP publique)
```
[quentin@router-tp1 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=11.2 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=118 time=12.3 ms
^C
--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 11.201/11.750/12.299/0.549 ms
```

- prouvez que vous pouvez résoudre des noms publics (ping d'un nom de domaine public)
```
[quentin@router-tp1 ~]$ ping ynov.com
PING ynov.com (104.26.11.233) 56(84) bytes of data.
64 bytes from 104.26.11.233 (104.26.11.233): icmp_seq=1 ttl=55 time=13.6 ms
64 bytes from 104.26.11.233 (104.26.11.233): icmp_seq=2 ttl=55 time=14.0 ms
64 bytes from 104.26.11.233 (104.26.11.233): icmp_seq=3 ttl=55 time=13.7 ms
^C
--- ynov.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 13.569/13.745/13.972/0.168 ms
```

☀️ **Accès internet LAN1 et LAN2**

- ajoutez une route par défaut sur les deux machines du LAN1
```
[quentin@node2-lan1-tp1 ~]$ sudo cat /etc/sysconfig/network-scripts/route-enp0s3
10.1.2.0/24 via 10.1.1.254 dev enp0s3
default via 10.1.1.254 dev enp0s3
```

- configurez l'adresse d'un serveur DNS que vos machines peuvent utiliser pour résoudre des noms
```
[quentin@node2-lan1-tp1 ~]$ sudo cat /etc/sysconfig/network-scripts/ifcfg-enp0s3

...

DNS=1.1.1.1
```

- prouvez que `node2.lan1.tp1` a un accès internet :
  - il peut ping une IP publique
  ```
  [quentin@node2-lan1-tp1 ~]$ ping 1.1.1.1
  PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
  64 bytes from 1.1.1.1: icmp_seq=1 ttl=54 time=13.4 ms
  64 bytes from 1.1.1.1: icmp_seq=2 ttl=54 time=14.9 ms
  ^C
  --- 1.1.1.1 ping statistics ---
  2 packets transmitted, 2 received, 0% packet loss, time 1002ms
  rtt min/avg/max/mdev = 13.434/14.166/14.898/0.732 ms
  ```

  - il peut ping un nom de domaine public
  ```
  [quentin@node2-lan1-tp1 ~]$ ping ynov.com
  PING ynov.com (104.26.11.233) 56(84) bytes of data.
  64 bytes from 104.26.11.233 (104.26.11.233): icmp_seq=1 ttl=54 time=13.8 ms
  64 bytes from 104.26.11.233 (104.26.11.233): icmp_seq=2 ttl=54 time=14.6 ms
  ^C
  --- ynov.com ping statistics ---
  2 packets transmitted, 2 received, 0% packet loss, time 1001ms
  rtt min/avg/max/mdev = 13.797/14.182/14.567/0.385 ms
  ```

# III. Services réseau

**Adresses IP et routage OK, maintenant, il s'agirait d'en faire quelque chose nan ?**

Dans cette partie, on va **monter quelques services orientés réseau** au sein de la topologie, afin de la rendre un peu utile que diable. Des machines qui se `ping` c'est rigolo mais ça sert à rien, des machines qui font des trucs c'est mieux.

## 1. DHCP

☀️ **Sur `dhcp.lan1.tp1`**

- setup du serveur DHCP

  - commande d'installation du paquet
  ```
  [quentin@dhcp-lan1-tp1 ~]$ sudo dnf -y install dhcp-server
  ```

  - fichier de conf
  ```
  [quentin@dhcp-lan1-tp1 ~]$ sudo cat /etc/dhcp/dhcpd.conf
  authoritative;

  option domain-name-servers 1.1.1.1;

  subnet 10.1.1.0 netmask 255.255.255.0 {
    range 10.1.1.100 10.1.1.200;
    option routers 10.1.1.254;
  }
  ```
  
  - service actif
  ```
  [quentin@dhcp-lan1-tp1 ~]$ sudo systemctl status dhcpd
  dhcpd.service - DHCPv4 Server Daemon
  Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; preset: disabled)
  Active: active (running) since Fri 2023-10-20 16:17:11 CEST; 57s ago
  ```

☀️ **Sur `node1.lan1.tp1`**

- demandez une IP au serveur DHCP
- prouvez que vous avez bien récupéré une IP *via* le DHCP
- prouvez que vous avez bien récupéré l'IP de la passerelle
[Capture wireshark récupération ip + paserelle](/captures/dhcp.pcapng)
- prouvez que vous pouvez `ping node1.lan2.tp1`
  ```
  [quentin@node1-lan1-tp1 ~]$ traceroute 10.1.2.12
  traceroute to 10.1.2.12 (10.1.2.12), 30 hops max, 60 byte packets
  1  _gateway (10.1.1.254)  0.638 ms  0.638 ms  0.617 ms
  2  10.1.2.12 (10.1.2.12)  16.871 ms !X  16.842 ms !X  16.806 ms !X
  ```

## 2. Web web web

☀️ **Sur `web.lan2.tp1`**

- n'oubliez pas de renommer la machine (`node2.lan2.tp1` devient `web.lan2.tp1`)
- setup du service Web
  - installation de NGINX
  - gestion de la racine web `/var/www/site_nul/`
  - configuration NGINX
  - service actif
  - ouverture du port firewall
- prouvez qu'il y a un programme NGINX qui tourne derrière le port 80 de la machine (commande `ss`)
```
[quentin@web-lan2-tp1 conf.d]$ sudo ss -tulnp | grep 80
tcp   LISTEN 0      511          0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=1936,fd=6),("nginx",pid=1935,fd=6))
tcp   LISTEN 0      511             [::]:80           [::]:*    users:(("nginx",pid=1936,fd=7),("nginx",pid=1935,fd=7))
```

- prouvez que le firewall est bien configuré
```
[quentin@web-lan2-tp1 conf.d]$ sudo firewall-cmd --list-all | grep 'services\|ports'
  services: cockpit dhcpv6-client http https ssh
  ports: 80/tcp
```

☀️ **Sur `node1.lan1.tp1`**

- éditez le fichier `hosts` pour que `site_nul.tp1` pointe vers l'IP de `web.lan2.tp1`
```
[quentin@node1-lan1-tp1 ~]$ sudo cat /etc/hosts | grep tp2
10.1.2.12 site_nul.tp2
```

- visitez le site nul avec une commande `curl` et en utilisant le nom `site_nul.tp1`
```
[quentin@node1-lan1-tp1 ~]$ curl http://site_nul.tp2
<p> Site web nul tp2 :) </p>
```