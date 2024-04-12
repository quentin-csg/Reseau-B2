# TP7 SECU : Accès réseau sécurisé

## Sommaire

- [TP7 SECU : Accès réseau sécurisé](#tp7-secu--accès-réseau-sécurisé)
  - [Sommaire](#sommaire)
- [I. VPN](#i-vpn)
- [II. SSH](#ii-ssh)
  - [1. Setup](#1-setup)
  - [3. Connexion par clé](#3-connexion-par-clé)
  - [4. Conf serveur SSH](#4-conf-serveur-ssh)
- [III. HTTP](#iii-http)
  - [1. Initial setup](#1-initial-setup)
  - [2. Génération de certificat et HTTPS](#2-génération-de-certificat-et-https)
    - [A. Préparation de la CA](#a-préparation-de-la-ca)
    - [B. Génération du certificat pour le serveur web](#b-génération-du-certificat-pour-le-serveur-web)
    - [C. Bonnes pratiques RedHat](#c-bonnes-pratiques-redhat)
    - [D. Config serveur Web](#d-config-serveur-web)

# I. VPN

| Machine            | LAN `10.7.1.0/24` | VPN `10.7.2.0/24` |
| ------------------ | ----------------- | ----------------- |
| `vpn.tp7.secu`     | `10.7.1.100/24`   |                   |
| `martine.tp7.secu` | `10.7.1.11/24`    | `10.7.2.11/24`    |
| ton PC             | X                 | `10.7.2.100/24`   |

🌞 **Monter un serveur VPN Wireguard sur `vpn.tp7.secu`**

```
[quentin@vpn ~]$ sudo cat /etc/sysctl.conf | tail -n 2
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
```
```
[quentin@vpn ~]$ sudo cat server.key
gGEmOjSy5HGktMoXY7DU/aeow/+8OvjC/7ebL/vFoGE=

[quentin@vpn ~]$ sudo cat server.pub
KKS7Gy/UCgrx1nxILCINL+wSaagqVz82Lrqkzsm6xgs=

[quentin@vpn ~]$ sudo cat clients/martine.key
CJDYtSepGjtniGsPgebzesqIFEyepq5qXXYI8Jx4WG4=

[quentin@vpn ~]$ sudo cat clients/martine.pub
RLwGjTaCycND7XvKoPPnSMpTIa4Od7Sjto002RRRJCo=
```

Serveur conf:
```
[quentin@vpn ~]$ sudo cat /etc/wireguard/wg0.conf
[sudo] password for quentin:
[Interface]
Address = 10.7.2.0/24
SaveConfig = false
PostUp = firewall-cmd --zone=public --add-masquerade
PostUp = firewall-cmd --add-interface=wg0 --zone=public
PostDown = firewall-cmd --zone=public --remove-masquerade
PostDown = firewall-cmd --remove-interface=wg0 --zone=public
ListenPort = 13337
PrivateKey = gGEmOjSy5HGktMoXY7DU/aeow/+8OvjC/7ebL/vFoGE=

[Peer]
PublicKey = RLwGjTaCycND7XvKoPPnSMpTIa4Od7Sjto002RRRJCo=
AllowedIPs = 10.7.2.11/32
```

Client martine conf:
```
[Interface]
Address = 10.7.2.11/24
PrivateKey = CJDYtSepGjtniGsPgebzesqIFEyepq5qXXYI8Jx4WG4=

[Peer]
PublicKey = KKS7Gy/UCgrx1nxILCINL+wSaagqVz82Lrqkzsm6xgs=
AllowedIPs = 0.0.0.0/0
Endpoint = 10.7.1.100:13337
```

Firewall vpn:
```
[quentin@vpn ~]$ sudo firewall-cmd --list-ports
13337/udp
```

Connexion des clients
```
[quentin@vpn ~]$ sudo wg show
interface: wg0
  public key: KKS7Gy/UCgrx1nxILCINL+wSaagqVz82Lrqkzsm6xgs=
  private key: (hidden)
  listening port: 13337

peer: RLwGjTaCycND7XvKoPPnSMpTIa4Od7Sjto002RRRJCo=
  endpoint: 10.7.1.11:34634
  allowed ips: 10.7.2.11/32
  latest handshake: 6 minutes, 14 seconds ago
  transfer: 10.83 KiB received, 9.83 KiB sent
```
```
[quentin@martine ~]$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=56 time=12.8 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=56 time=13.4 ms
^C
--- 1.1.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 12.848/13.130/13.413/0.282 ms
```

🌞 **Client Wireguard sur votre PC**

```
[quentin@vpn ~]$ sudo cat /etc/wireguard/clients/pc.key
IKpqEf/j3yvjHxu4UjaezlzJrRG9TRRScJpTx70eIkA=

[quentin@vpn ~]$ sudo cat /etc/wireguard/clients/pc.pub
DvH5N4RqeekUBWZsPywwIWG9G3YvDs0oT4wp9fgqhBw=
```

Conf vpn en rajoutant notre pc en tant que client:
```
[quentin@vpn ~]$ sudo cat /etc/wireguard/wg0.conf | tail -n 3
[Peer]
PublicKey = DvH5N4RqeekUBWZsPywwIWG9G3YvDs0oT4wp9fgqhBw=
AllowedIPs = 17.7.2.100/32
```

Conf pc
```
[Interface]
Address = 10.7.2.100/24
PrivateKey = IKpqEf/j3yvjHxu4UjaezlzJrRG9TRRScJpTx70eIkA=

[Peer]
PublicKey = KKS7Gy/UCgrx1nxILCINL+wSaagqVz82Lrqkzsm6xgs=
AllowedIPs = 0.0.0.0/0
Endpoint = 10.7.1.100:13337
```

Ping vers martine
```
PS C:\Users\qcass> ping 10.7.1.11

Envoi d’une requête 'Ping'  10.7.1.11 avec 32 octets de données :
Réponse de 10.7.1.11 : octets=32 temps<1ms TTL=64
Réponse de 10.7.1.11 : octets=32 temps<1ms TTL=64
Réponse de 10.7.1.11 : octets=32 temps<1ms TTL=64

Statistiques Ping pour 10.7.1.11:
    Paquets : envoyés = 3, reçus = 3, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 0ms, Maximum = 0ms, Moyenne = 0ms
```

# II. SSH

## 1. Setup

| Machine            | LAN `10.7.1.0/24` | VPN `10.7.2.0/24` |
| ------------------ | ----------------- | ----------------- |
| `vpn.tp7.secu`     | `10.7.1.100/24`   |                   |
| `martine.tp7.secu` | `10.7.1.11/24`    | `10.7.2.11/24`    |
| `bastion.tp7.secu` | `10.7.1.12/24`    | `10.7.2.12/24`    |
| `web.tp7.secu`     | `10.7.1.13/24`    | `10.7.2.13/24`    |
| ton PC             | X                 | `10.7.2.100/24`   |

🌞 **Générez des confs Wireguard pour tout le monde**

```
[quentin@vpn ~]$ wg genkey | sudo tee /etc/wireguard/clients/bastion.key
UHAI9USJxShckhHk+VrWUtRz97nanciOfXEzNVeNsXU=

[quentin@vpn ~]$ sudo cat /etc/wireguard/clients/bastion.key | wg pubkey | sudo tee /etc/wireguard/clients/bastion.pub
mAgkQx6L131aat4iRH0ctk7U/tIfFpwfOEnSjztUtgE=

[quentin@vpn ~]$ wg genkey | sudo tee /etc/wireguard/clients/web.key
UCRTZKQcYDvlJYzd3hi89WsK83RKup0GJhcqoVUcjXo=

[quentin@vpn ~]$ sudo cat /etc/wireguard/clients/web.key | wg pubkey | sudo
tee /etc/wireguard/clients/web.pub
FOagJb3jI2MlDarrKaH1EzU1+ppXdLX+ih8r1ljf3Hw=
```

## 3. Connexion par clé

🌞 **Générez une nouvelle paire de clés pour ce TP**

Utilisation de ECDSA comme algorithme de signature numérique. Il fonctionne sur la représentation mathématique des courbes elliptiques.
La complexité des courbes elliptiques fait de l'ECDSA une méthode plus complexe que le RSA.
Pour atteindre un niveau de sécurité de 112 bits, il faut une clé de longueurs 2048 bits avec l'algorithme RSA contre seulement 224 pour l'ECDSA donc cela offre de meilleures performances avec une taille de clé plus petite.

Création clé pub/priv sur martine pour pouvoir se connecter en ssh sur le bastion
```
[quentin@martine ~]$ ssh-keygen -t ecdsa -f $HOME/.ssh/ecdsa
Generating public/private ecdsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/quentin/.ssh/ecdsa
Your public key has been saved in /home/quentin/.ssh/ecdsa.pub
The key fingerprint is:
SHA256:vySztLiI9BMxdzHe2fZ9c5+bptLNFK7xkqTLciGGkGs quentin@martine.tp7.secu
The key's randomart image is:
+---[ECDSA 256]---+
|                 |
|        o        |
|      .. + o     |
|    oo. o o o  . |
|     +o.S  . ....|
|    .E . + . o.++|
|  . ..  = + = O *|
| . o.. o B.= = *o|
|  . o.o.o +oo.+o.|
+----[SHA256]-----+
```

Modification de la config du bastion pour ne plus accepter les connexion par mot de passe et ajout de la clé publique de martine:
```
[quentin@bastion ~]$ sudo cat /etc/ssh/sshd_config | grep 'PasswordAuthentication no' && sudo cat /etc/ssh/sshd_config | grep PubkeyAuthentication
PasswordAuthentication no
PubkeyAuthentication yes
```

Clé publique de martine dans le fichier authorized_keys:
```
[quentin@bastion ~]$ sudo cat .ssh/authorized_keys
ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMJjT4CYpM9GrR3jZPBxkcXLWF1blnpJKKEa5Fs4XzMQGABFzYLVVv50eAdSHT6aBJtDTMmYGPhRM2ySdKNrOGE= quentin@martine.tp7.secu
```

Connexion vers le bastion en utilisant plus le mot de passe:
```
[quentin@martine ~]$ ssh -i $HOME/.ssh/ecdsa quentin@10.7.2.12
Enter passphrase for key '/home/quentin/.ssh/ecdsa':
Last login: Sun Dec 17 13:06:26 2023 from 10.7.2.0
```
Faire la même chose de web vers bastion, et ensuite de bastion vers web et martine pour exécuter le jump pour la connexion entre martine et web

## 4. Conf serveur SSH

🌞 **Changez l'adresse IP d'écoute**


- vos serveurs SSH ne doivent être disponibles qu'au sein du réseau VPN
- prouvez que vous ne pouvez plus accéder à une sesion SSH en utilisant l'IP host-only (obligé de passer par le VPN)

Fichier de conf de web:
```
[quentin@web ~]$ sudo cat /etc/ssh/sshd_config | grep 10.7.2.13
ListenAddress 10.7.2.13
```

SSH de bastion vers web refusé si on utilise l'IP host-only
```
[quentin@bastion ~]$ ssh -i $HOME/.ssh/ecdsa-web quentin@10.7.1.13 -p 8081
ssh: connect to host 10.7.1.13 port 8081: Connection refused
[quentin@bastion ~]$ ssh -i $HOME/.ssh/ecdsa-web quentin@10.7.2.13 -p 8081
Last login: Sun Dec 17 12:17:06 2023 from 10.7.1.1
```

🌞 **Améliorer le niveau de sécurité du serveur**

- mettre en oeuvre au moins 3 configurations additionnelles pour améliorer le niveau de sécurité

Empêcher les connexion en tant qu'utilisateur root
```
PermitRootLogin no
```

Empêcher les attaques par bruteforce en mettant en limitant le nombre de tentatives possibles et en ajoutant un délai après un certains nombres de tentatives échouées
```
MaxAuthTries 2
LoginGraceTime 30
```

Mettre un autre port que le port par défaut
```
Port 8081
```

# III. HTTP

## 1. Initial setup

🌞 **Monter un bête serveur HTTP sur `web.tp7.secu`**

```
[quentin@web nginx]$ cat /var/www/site_nul/html/index.html
<!doctype html>
<html lang="fr">

<head>
  <meta charset="utf-8">
  <title>MEOW</title>
  <link rel="stylesheet" href="style.css">
  <script src="script.js"></script>
</head>

<body>
  <h1>MEOW site nul</h1>
</body>

</html>
```
```
[quentin@web nginx]$ cat /etc/nginx/conf.d/site_nul.conf
server {
    server_name web.tp7.secu;

    listen 10.7.2.13:80;

    # vous collez un ptit index.html dans ce dossier et zou !
    root /var/www/site_nul/html;
}
```

On met en commentaire le site par défaut pour qu'il ne puissent plus être utilisé.
```
[quentin@web nginx]$ cat nginx.conf | grep -A 15 'server {'
#    server {
#        listen       80;
#        listen       [::]:80;
#        server_name  _;
#        root         /usr/share/nginx/www/html;

        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;

#        error_page 404 /404.html;
#        location = /404.html {
#        }

#        error_page 500 502 503 504 /50x.html;
#        location = /50x.html {
#        }
```
🌞 **Site web joignable qu'au sein du réseau VPN**

- le site web ne doit écouter que sur l'IP du réseau VPN
```
[quentin@bastion ~]$ curl 10.7.2.13
<!doctype html>
<html lang="fr">

<head>
  <meta charset="utf-8">
  <title>MEOW</title>
  <link rel="stylesheet" href="style.css">
  <script src="script.js"></script>
</head>

<body>
  <h1>MEOW site nul</h1>
</body>

</html>
```

- le trafic à destination du port 80 n'est autorisé que si la requête vient du réseau VPN (firewall)
```
[quentin@web nginx]$ sudo firewall-cmd --list-rich-rule
rule family="ipv4" source address="10.7.2.0/24" port port="80" protocol="tcp" accept
```

- prouvez qu'il n'est pas possible de joindre le site sur son IP host-only
```
[quentin@bastion ~]$ curl 10.7.1.13
curl: (7) Failed to connect to 10.7.1.13 port 80: Connection refused
```

## 2. Génération de certificat et HTTPS

### A. Préparation de la CA

🌞 **Générer une clé et un certificat de CA**

```
[quentin@web ~]$ sudo openssl genrsa -des3 -out CA.key 4096
[sudo] password for quentin:
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:

[quentin@web ~]$ sudo openssl req -x509 -new -nodes -key CA.key -sha256 -days 1024  -out CA.pem
Enter pass phrase for CA.key:
...
```

### B. Génération du certificat pour le serveur web

🌞 **Générer une clé et une demande de signature de certificat pour notre serveur web**

```
[quentin@web ~]$ openssl req -new -nodes -out web.tp7.secu.csr -newkey rsa:4096 -keyout web.tp7.secu.key
..+...+....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..........+..+.+..+....+...........+...............+...+.+...+.....+......+.+......+...+.....+.........+......+.+.....+....+.....+....+.....+.+.........+...........+.......+...+......+......+.........+......+........+.+..............+.+......+........+......+...+...+.......+...........+....+...+..+.......+...+..+....+......+........+.+...........+.+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*...+..........+.......................+......+......+....+......+......+..+......+......+.............+..+.+.....+.........+....+..+.........+.........+....+...........+....+.....+............+...+..........+..+.+..............+.+.....+...+.......+..+.......+...+.....+...+..........+...+..+.......+...+..+.......+.....+......+...+................+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
..+...+.........+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.............+.......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..+...+...+..+.......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----
You are about ...
```

🌞 **Faire signer notre certificat par la clé de la CA**

- préparez un fichier `v3.ext` qui contient :
```
[quentin@web ~]$ cat v3.ext
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = web.tp7.secu
DNS.2 = www.tp7.secu
```

- effectuer la demande de signature pour récup un certificat signé par votre CA :
```
[quentin@web ~]$ sudo openssl x509 -req -in web.tp7.secu.csr -CA CA.pem -CAkey CA.key -CAcreateserial -out web.tp7.secu.crt -days 500 -sha256 -extfile v3.ext
Certificate request self-signature ok
subject=C = fr, L = Default City, O = Default Company Ltd
Enter pass phrase for CA.key:
```

### C. Bonnes pratiques RedHat

🌞 **Déplacer les clés et les certificats dans l'emplacement réservé**

- gérez correctement les permissions de ces fichiers

Dossier des certificats
```
[quentin@web certs]$ ls -la CA.pem web.tp7.secu.crt
-rw-r--r-- 1 root    root    1931 Dec 17 22:01 CA.pem
-rw-r--r-- 1 quentin quentin 1996 Dec 17 22:08 web.tp7.secu.crt
```

Dossier des clés
```
[quentin@web private]$ ls -la
total 8
drwxr-xr-x. 2 root    root      44 Dec 17 22:22 .
drwxr-xr-x. 5 root    root     126 Nov 28 15:14 ..
-r--------  1 root    root    3422 Dec 17 22:01 CA.key
-r--------  1 quentin quentin 3272 Dec 17 22:05 web.tp7.secu.key
```
### D. Config serveur Web

🌞 **Ajustez la configuration NGINX**

- le site web doit être disponible en HTTPS en utilisant votre clé et votre certificat
- une conf minimale ressemble à ça :

```nginx
server {
    server_name web.tp7.secu;

    listen 10.7.1.103:443 ssl;

    ssl_certificate /etc/pki/tls/certs/web.tp7.secu.crt;
    ssl_certificate_key /etc/pki/tls/private/web.tp7.secu.key;
    
    root /var/www/site_nul;
}
```

🌞 **Prouvez avec un `curl` que vous accédez au site web**

- depuis votre PC
```
┌──(quentin㉿debian)-[~/wireguard]
└─$ curl -k https://10.7.2.13
<!doctype html>
<html lang="fr">

<head>
  <meta charset="utf-8">
  <title>MEOW</title>
  <link rel="stylesheet" href="style.css">
  <script src="script.js"></script>
</head>

<body>
  <h1>MEOW site nul</h1>
</body>

</html>
```