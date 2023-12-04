# TP1 : Maîtrise réseau du poste

## I. Basics

☀️ **Carte réseau WiFi**

Déterminer...

- l'adresse MAC de votre carte WiFi
````
ipconfig /all

Carte réseau sans fil Wi-Fi :
Adresse physique . . . . . . . . . . . : 60-E3-2B-90-26-E4 
````

- l'adresse IP de votre carte WiFi
````
Adresse IPv4. . . . . . . . . . . . . .: 10.33.76.182
````

- le masque de sous-réseau du réseau LAN auquel vous êtes connectés en WiFi
  - en notation CIDR, par exemple `/16`
  ````
  Masque de sous-réseau. . . . . . . . . : 255.255.240.0/16
  ````

  - ET en notation décimale, par exemple `255.255.0.0`
  ````
  Masque de sous-réseau. . . . . . . . . : 255.255.240.0
  ````

---

☀️ **Déso pas déso**

- l'adresse de réseau du LAN auquel vous êtes connectés en WiFi
````
10.33.64.0
````

- l'adresse de broadcast
````
10.33.79.255
````

- le nombre d'adresses IP disponibles dans ce réseau
````
4,096
````

---

☀️ **Hostname**

- déterminer le hostname de votre PC
````
hostname
LAPTOP-4K0GL7E4
````

---

☀️ **Passerelle du réseau**

Déterminer...

- l'adresse IP de la passerelle du réseau
````
ipconfig /all

Passerelle par défaut. . . . . . . . . : 10.33.79.254
````

- l'adresse MAC de la passerelle du réseau
````
arp -a

Interface : 10.33.76.182 --- 0xc
10.33.79.254          7c-5a-1c-d3-d8-76
````

---

☀️ **Serveur DHCP et DNS**

Déterminer...

- l'adresse IP du serveur DHCP qui vous a filé une IP
```
ipconfig /all

Serveur DHCP . . . . . . . . . . . . . : 10.33.79.254
```

- l'adresse IP du serveur DNS que vous utilisez quand vous allez sur internet
```
ipconfig /all

Serveurs DNS. . .  . . . . . . . . . . : 8.8.8.8
```

---

☀️ **Table de routage**

Déterminer...

- dans votre table de routage, laquelle est la route par défaut
```
route print

IPv4 Table de routage
==========================
Itinéraires actifs :
10.33.79.254
```

# II. Go further


☀️ **Hosts ?**

- prouvez avec un `ping b2.hello.vous` que ça ping bien `1.1.1.1`
```
ping b2.hello.vous

Envoi d’une requête 'ping' sur b2.hello.vous [1.1.1.1] avec 32 octets de données :
Réponse de 1.1.1.1 : octets=32 temps=11 ms TTL=57
Réponse de 1.1.1.1 : octets=32 temps=10 ms TTL=57

Statistiques Ping pour 1.1.1.1:
    Paquets : envoyés = 2, reçus = 2, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 10ms, Maximum = 11ms, Moyenne = 10ms
Ctrl+C
^C
```

> Vous pouvez éditer en GUI, et juste me montrer le contenu du fichier depuis le terminal pour le compte-rendu.
```
C:\Windows\System32\drivers\etc>type hosts

# localhost name resolution is handled within DNS itself.
#       127.0.0.1       localhost
#       ::1             localhost
1.1.1.1 b2.hello.vous
```

---

☀️ **Go mater une vidéo youtube et déterminer, pendant qu'elle tourne...**


```
netstat -a -b -n
[chrome.exe]
  TCP    10.33.76.182:50067     172.217.18.195:443     ESTABLISHED
```

- l'adresse IP du serveur auquel vous êtes connectés pour regarder la vidéo
```
172.217.18.195
```

- le port du serveur auquel vous êtes connectés
```
443
```

- le port que votre PC a ouvert en local pour se connecter au port du serveur distant
```
50067
```

---

☀️ **Requêtes DNS**

Déterminer...

- à quelle adresse IP correspond le nom de domaine `www.ynov.com`
```
nslookup www.ynov.com

104.26.11.233
104.26.10.233
172.67.74.226
```

- à quel nom de domaine correspond l'IP `174.43.238.89`
```
nslookup 174.43.238.89

Nom :    89.sub-174-43-238.myvzw.com
```

---

☀️ **Hop hop hop**

Déterminer...

- par combien de machines vos paquets passent quand vous essayez de joindre `www.ynov.com`
```
tracert www.ynov.com

Détermination de l’itinéraire vers www.ynov.com [172.67.74.226]
avec un maximum de 30 sauts :

  1     1 ms     2 ms     1 ms  10.33.79.254
  2     4 ms     2 ms     2 ms  145.117.7.195.rev.sfr.net [195.7.117.145]
  3     4 ms     3 ms     3 ms  237.195.79.86.rev.sfr.net [86.79.195.237]
  4     4 ms     2 ms     3 ms  196.224.65.86.rev.sfr.net [86.65.224.196]
  5    13 ms    10 ms    10 ms  12.148.6.194.rev.sfr.net [194.6.148.12]
  6    32 ms    10 ms    14 ms  12.148.6.194.rev.sfr.net [194.6.148.12]
  7    12 ms    11 ms    11 ms  141.101.67.48
  8    14 ms    12 ms    11 ms  172.71.124.4
  9    13 ms    12 ms    11 ms  172.67.74.226```
```

---

☀️ **IP publique**

Déterminer...

- l'adresse IP publique de la passerelle du réseau (le routeur d'YNOV donc si vous êtes dans les locaux d'YNOV quand vous faites le TP)

```
nslookup myip.opendns.com resolver1.opendns.com


Réponse ne faisant pas autorité :
Nom :    myip.opendns.com
Address:  37.174.206.252
```

---

☀️ **Scan réseau**

Déterminer...

- combien il y a de machines dans le LAN auquel vous êtes connectés

```
arp -a

Interface : 10.33.76.182 --- 0xc
  Adresse Internet      Adresse physique      Type
  10.33.65.188          32-fd-2f-fc-81-4a     dynamique
  10.33.68.216          f4-6d-3f-32-3f-5c     dynamique
  10.33.69.0            f8-89-d2-7c-36-7d     dynamique
  10.33.69.3            e0-2b-e9-77-20-bd     dynamique
  10.33.70.194          30-89-4a-d2-5a-aa     dynamique
  10.33.71.128          a0-29-42-24-b0-3f     dynamique
  10.33.71.150          10-a5-1d-85-3a-90     dynamique
  10.33.72.155          70-a8-d3-1d-13-3e     dynamique
  10.33.73.67           40-1a-58-4b-44-84     dynamique
  10.33.75.207          f8-ac-65-0f-5b-9f     dynamique
  10.33.76.245          70-66-55-a3-8b-a1     dynamique
  10.33.77.4            40-ec-99-aa-4c-ea     dynamique
  10.33.79.254          7c-5a-1c-d3-d8-76     dynamique
  10.33.79.255          ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  230.0.0.1             01-00-5e-00-00-01     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique
```


# III. Le requin

☀️ **Capture ARP**

- 📁 fichier `arp.pcap`
- capturez un échange ARP entre votre PC et la passerelle du réseau

[Avec filtre ARP](/captures/arp.pcapng)

---

☀️ **Capture DNS**

- 📁 fichier `dns.pcap`
- capturez une requête DNS vers le domaine de votre choix et la réponse
- vous effectuerez la requête DNS en ligne de commande

[Avec filtre DNS](/captures/dns.pcapng)

---

☀️ **Capture TCP**

- 📁 fichier `tcp.pcap`
- effectuez une connexion qui sollicite le protocole TCP
- je veux voir dans la capture :
  - un 3-way handshake
  - un peu de trafic
  - la fin de la connexion TCP

[Avec filtre TCP](/captures/tcp.pcapng)