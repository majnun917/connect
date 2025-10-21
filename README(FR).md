<h1 align="center">Connect</h1>


Ce projet consiste à configurer la connectivité réseau entre trois machines virtuelles (VM).
On a juste acces à une seule machine, la ``machine 2``. Cette dernière se connectera à Internet grâce à la machine ``box``, qui jouera le rôle de routeur. La ``box``, quant à elle, accèdera à Internet via un réseau [NAT(Network Address Translation)](https://en.wikipedia.org/wiki/Network_address_translation).

Pour que les machines puissent communiquer, elles doivent disposer d'une [adresse IP](https://en.wikipedia.org/wiki/IP_address).
Les adresses IP peuvent être configurées de manière ``statique`` (manuellement) ou obtenues de façon ``dynamique`` auprès d'un [serveur DHCP](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol).


L'image suivante illustre la configuration des machines:    

<img src="config.png" width="60%">        


**Utilisation dans le projet** :
- INT_1 : Réseau entre box, machine1, machine2
- INT_2 : Réseau entre box et NAT


**Rôles** :
- **Box** : Routeur avec NAT, serveur DHCP, DNS
- **machine1** : Client avec IP statique
- **machine2** : Client que vous configurez en DHCP


On commence par enumerer les etapes de la configuration des VM, ensuite quelques commandes qui permettent de gerer les conflits car si vous remarque sur l'image on a les addresses IP sur les 2VM clients et enfin expliquer quelques concepts utilises dans ce projet.

## Les etapes de la configuration dans l'interface de VirtualBox     
1. Configurer la box en allant sur Settings->Network
    - Configurer les 2 NIC:
      * Adapter 1: cocher Enabale Network, choisis ``Internal Network`` et le nom: ``intnet(ex)``
      * Adapter 2: cocher Enabale Network et choisis le ``NAT``
      * Et appuyer sur ``OK``.
2. Configurer les 2 VM, en allant sur Settings->Network:
    - Ici on a ctive juste Adapter 1, en choisissant ``Internal Network`` et le meme nom pour la box: ``intnet(ex)``
    - Et appuyer sur ``OK``.

## Les commandes        
Apres avoir configurer les machines, on peut verifier l'addresse IP actuelle (``machine 2``) par:
```sh
ip addr show
```     
Et s'il y'a des conflits dus aux adresses IP identiques. On change l'adresse d'une machine.
Il faut savoir que Linux offre plusieurs méthodes pour configurer le réseau.
Ici on peut changer l'addresse IP par deux methodes:
- Methode Static: on edite manuellement le fichier  /etc/network/interfaces:
```sh
nano /etc/network/interfaces
```     
On specifie l'adresse IP:     

``` sh 
    auto enps03(interface name)
    iface enps03 inet static
    address 192.0.2.3   # Ici on a changé .2 par .3
    gateway 192.168.0.1
    dns-namserver 8.8.8.8 
```         
Et on peut redemarrer l'interface par la commande suivante:
```sh
ifdown enps03 && ifup enps03
```
ou de redemarrer la VM.
- Methode dynamique: Toujours par l'edition du fichier  /etc/network/interfaces, mais ici on ne change pas manuellement l'addresse:

```sh
nano /etc/network/interfaces
```     
Et modifie par:     

```sh 
    auto enps03
    iface enps03 inet dhcp
```

Verifie toujours le l'addresse actuelle par : ``ip addr show``

## Les Concepts     

### Adresse IP (Internet Protocol)

Une **adresse IP** est un identifiant unique attribué à chaque appareil connecté à un réseau.

Ici on prend l'exemple de **IPv4** mais au niveau des liens importants, on a l'explication et la difference **IPv4** et **IPv6**.
**Format IPv4** : 4 octets (32 bits) écrits en notation décimale pointée
- Exemple : `192.168.0.2`
- Chaque octet : 0-255

**Types d'adresses IP** :

1. **Adresses Publiques** : Routables sur Internet (ex: `8.8.8.8`)
2. **Adresses Privées** : Non routables sur Internet, réservées aux réseaux locaux
   - Classe A : `10.0.0.0` - `10.255.255.255`
   - Classe B : `172.16.0.0` - `172.31.255.255`
   - Classe C : `192.168.0.0` - `192.168.255.255`

### Masque de Sous-réseau

Le **masque de sous-réseau** définit quelle partie de l'adresse IP identifie le réseau et quelle partie identifie l'hôte.

**Exemples** :
- `255.255.255.0` ou `/24` : Les 3 premiers octets = réseau, dernier octet = hôte
- `255.255.0.0` ou `/16` : Les 2 premiers octets = réseau
- `255.0.0.0` ou `/8` : Le premier octet = réseau

**Dans notre projet** :
- Réseau : `192.168.0.0/24`
- Adresses disponibles : `192.168.0.1` à `192.168.0.254`
- `192.168.0.0` : Adresse réseau
- `192.168.0.255` : Adresse broadcast

### Conflit d'Adresses IP

Un **conflit IP** se produit quand deux appareils ont la même adresse IP sur le même réseau.

**Conséquences** :
- Perte de paquets importante
- Connexions instables
- Les deux machines se "battent" pour répondre aux requêtes
- Comportement imprévisible du réseau

**Dans notre projet** : machine1 et machine2 ont toutes deux `192.168.0.2`

## DHCP (Dynamic Host Configuration Protocol)

Le **DHCP** est un protocole qui attribue automatiquement des configurations réseau aux appareils.

**Éléments attribués par DHCP** :
- Adresse IP
- Masque de sous-réseau
- Passerelle par défaut (Gateway)
- Serveurs DNS
- Durée de bail (Lease Time)

### Fonctionnement de DHCP (Processus DORA)

1. **DISCOVER** : Le client envoie un broadcast "Je cherche un serveur DHCP"
2. **OFFER** : Le serveur DHCP répond "Voici une IP disponible"
3. **REQUEST** : Le client demande "Je veux cette IP"
4. **ACKNOWLEDGE** : Le serveur confirme "OK, cette IP est à toi"

### Serveur vs Client DHCP

**Serveur DHCP** :
- Gère un pool d'adresses IP disponibles
- Attribue et suit les adresses distribuées
- Dans notre projet : la "box" est le serveur DHCP

**Client DHCP** :
- Demande une configuration réseau au serveur
- Renouvelle son bail périodiquement
- Dans notre projet : machine2 devient un client DHCP

### IP Statique vs Dynamique

**IP Statique** :
- Configurée manuellement
- Ne change jamais
- Avantages : Prévisible, stable pour les serveurs
- Inconvénients : Risque de conflits, gestion manuelle

**IP Dynamique (DHCP)** :
- Attribuée automatiquement
- Peut changer à l'expiration du bail
- Avantages : Pas de conflits, gestion automatisée
- Inconvénients : Adresse peut changer

## NAT (Network Address Translation)

### Principe du NAT

Le **NAT** traduit des adresses IP privées en adresses IP publiques pour permettre l'accès à Internet.Il est le moyen le plus simple d'accéder à un réseau externe à partir d'une machine virtuelle. En général, elle ne nécessite aucune configuration sur le réseau hôte et le système invité. C'est pourquoi il s'agit du mode réseau par défaut dans Oracle VM VirtualBox.

**Pourquoi NAT ?** :
- Pénurie d'adresses IPv4 publiques
- Sécurité : Les machines internes sont masquées
- Permet à plusieurs appareils de partager une seule IP publique

### 3.2 Types de NAT

**1. NAT Statique (Static NAT)** :
- Une IP privée = Une IP publique fixe
- Utilisé pour les serveurs accessibles depuis Internet

**2. NAT Dynamique (Dynamic NAT)** :
- Pool d'IP publiques partagées
- Attribution dynamique

**3. PAT (Port Address Translation) / NAT Overload** :
- Plusieurs IP privées = Une seule IP publique
- Utilise des numéros de port différents
- **C'est ce qu'utilise notre box !**

### 3.3 Fonctionnement du NAT dans notre Projet

```
machine2 (192.168.0.2) → Box → Internet (10.0.2.2) → Internet
```

**Étapes** :
1. machine2 envoie un paquet vers `google.com`
2. Source : `192.168.0.2:54321`, Destination : `142.250.x.x:80`
3. La box traduit : Source : `10.0.2.2:54321`, Destination : `142.250.x.x:80`
4. Google répond à `10.0.2.2:54321`
5. La box traduit et renvoie à `192.168.0.2:54321`

### Table NAT

La box maintient une **table NAT** qui mappe :
- IP privée : Port → IP publique : Port

Exemple :
```
192.168.0.2:54321 ↔ 10.0.2.2:54321
192.168.0.3:43210 ↔ 10.0.2.2:43210
```

## Routage et Tables de Routage


Le **routage** est le processus de sélection du chemin pour acheminer les paquets vers leur destination.

### Passerelle par Défaut (Default Gateway)

La **passerelle** est le routeur qui permet d'accéder aux réseaux extérieurs.

**Dans notre projet** :
- Passerelle de machine2 : `192.168.0.1` (la box)
- Tout le trafic Internet passe par cette passerelle

### Table de Routage

La **table de routage** contient les règles de routage d'un système.

```sh
# Afficher la table de routage
ip route show
```

**Exemple de sortie** :
```
default via 192.168.0.1 dev enps03      # Tout le reste → passerelle
192.168.0.0/24 dev enps03 proto kernel  # Réseau local → directement
```

**Signification** :
- `default` : Route par défaut (pour tout ce qui n'est pas local)
- `via 192.168.0.1` : Passer par cette passerelle
- `dev enps03` : Utiliser cette interface
- `192.168.0.0/24` : Réseau local, communication directe

### Routage dans le Projet

```
machine2 → ping google.com
↓
Table de routage : "Pas dans 192.168.0.0/24 → utiliser default gateway"
↓
Paquet envoyé à 192.168.0.1 (box)
↓
Box applique NAT et route vers Internet
```


## DNS (Domain Name System)

Le **DNS** traduit les noms de domaine en adresses IP.

**Exemple** :
- Vous tapez : `google.com`
- DNS traduit : `142.250.185.46`

### Hiérarchie DNS

```
. (root)
  ↓
.com (TLD - Top Level Domain)
  ↓
google.com (Domain)
  ↓
www.google.com (Subdomain)
```

### Configuration DNS sous Linux

```sh
# Fichier de configuration DNS
cat /etc/resolv.conf

# Exemple de contenu :
nameserver 192.168.0.1   # DNS de la box
nameserver 8.8.8.8       # DNS Google (backup)
```


### DNS dans notre Projet

```
machine2 : ping google.com
↓
1. Consulte /etc/resolv.conf → DNS server: 192.168.0.1
↓
2. Demande à la box : "Quelle est l'IP de google.com ?"
↓
3. Box (ou DNS en amont) répond : "142.250.185.46"
↓
4. machine2 ping 142.250.185.46
```

## Protocole ICMP et Commande Ping

### Protocole ICMP

**ICMP** (Internet Control Message Protocol) est un protocole de diagnostic et de contrôle.

**Utilisations** :
- Tests de connectivité (ping)
- Messages d'erreur (destination inaccessible)
- Tracer les routes (traceroute)

**ICMP n'est pas TCP ou UDP** : C'est un protocole de couche 3 (réseau)

### Commande Ping

Le **ping** envoie des requêtes ICMP Echo Request et attend des Echo Reply.

```sh
# Ping simple
ping google.com

# Ping avec limite de paquets(ic 4 paquets)
ping -c 4 google.com

# Ping avec timeout
timeout 1m ping google.com
ou
timeout --signal SIGINT 1m ping google.com
```

**Sortie du Ping** :
```
64 bytes from google.com (142.250.185.46): icmp_seq=1 ttl=117 time=15.2 ms
```

- `64 bytes` : Taille du paquet
- `icmp_seq` : Numéro de séquence
- `ttl` : Time To Live (nombre de sauts restants)
- `time` : Latence (Round Trip Time)

### Interprétation des Résultats

**Statistiques du Ping** :
```
--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 14.2/15.1/16.3/0.8 ms
```

- `0% packet loss` : Connexion stable
- `>10% packet loss` : Problème réseau (conflit IP dans notre cas)
- `100% packet loss` : Pas de connectivité

**Temps de Réponse (RTT)** :
- `< 1ms` : Réseau local
- `1-30ms` : Très bon 
- `30-100ms` : Acceptable
- `> 100ms` : Lent
- `> 300ms` : Très lent

### Perte de Paquets et Conflit IP

**Dans notre projet** :
- **Avec conflit** : machine1 et machine2 ont la même IP
  - Quand un paquet arrive pour `192.168.0.2`, qui répond ?
  - Les deux machines essaient de répondre
  - Résultat : perte de paquets, réponses manquantes
  
- **Après résolution** : IPs différentes
  - Chaque paquet arrive à la bonne machine
  - Pas de confusion
  - Perte de paquets < 1%


## Interfaces Réseau


Une **interface réseau** est un point de connexion entre un ordinateur et un réseau.

**Types** :
- **Physique** : Carte réseau Ethernet (enps03, enp0s3)
- **Virtuelle** : Loopback (lo), tunnels, VPN
- **Sans fil** : WiFi (wlan0, wlp2s0)

### Nomenclature des Interfaces

**Ancienne nomenclature** :
- `eth0`, `eth1` : Ethernet
- `wlan0` : WiFi
- `lo` : Loopback

**Nouvelle nomenclature (Systemd)** :
- `enp0s3` : en=Ethernet, p0=bus PCI 0, s3=slot 3(notre cas ici)
- `wlp2s0` : wl=Wireless LAN
- `eno1` : on-board device index 1

### Interface Loopback

**lo (127.0.0.1)** : Interface virtuelle locale
- Permet à la machine de communiquer avec elle-même
- Toujours active
- Utilisée pour tester les services locaux


### États d'une Interface

```sh
# Sortie de ip link show :
2: enps03: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
```

- `UP` : Interface activée
- `LOWER_UP` : Câble connecté (couche physique OK)
- `BROADCAST` : Supporte le broadcast
- `MULTICAST` : Supporte le multicast
- `mtu 1500` : Maximum Transmission Unit (taille max des paquets)


### Autres Méthodes de Configuration

N'oubliez pas linux offre plusieurs méthodes pour configurer le réseau :

1. **Commande ip** (temporaire)
2. **dhclient** (DHCP)
3. **/etc/network/interfaces** (Debian/Ubuntu)

### Configuration Temporaire (ip)

```sh
# Assigner une IP manuellement
ip addr add 192.168.0.10/24 dev enps03

# Supprimer une IP
ip addr del 192.168.0.10/24 dev enps03

# Ajouter une route par défaut
ip route add default via 192.168.0.1

# Activer l'interface
ip link set enps03 up
```

**Note** : Ces changements sont perdus au redémarrage !


**Redémarrer le réseau** :
```sh
systemctl restart networking
# ou
/etc/init.d/networking restart
```



### 8.6 Fichiers de Configuration Importants

```sh
# Configuration DNS
/etc/resolv.conf

# Configuration réseau (Debian style)
/etc/network/interfaces

# Configuration Netplan
/etc/netplan/*.yaml

# Hostname
/etc/hostname
/etc/hosts

# Baux DHCP
/var/lib/dhcp/dhclient.leases

# Logs réseau
/var/log/syslog
journalctl -u networking
```

## 9. Modèle OSI et TCP/IP

### 9.1 Modèle OSI (7 couches)

Le **modèle OSI** décrit comment les données circulent dans un réseau :

| Couche | Nom | Rôle | Exemples |
|--------|-----|------|----------|
| 7 | Application | Interface utilisateur | HTTP, FTP, SSH, DNS |
| 6 | Présentation | Format des données | SSL/TLS, JPEG, ASCII |
| 5 | Session | Gestion des sessions | NetBIOS, RPC |
| 4 | Transport | Transmission fiable | TCP, UDP |
| 3 | Réseau | Routage | IP, ICMP, NAT |
| 2 | Liaison | Adressage physique | Ethernet, MAC, Switch |
| 1 | Physique | Signal électrique | Câbles, WiFi, Bits |

### 9.2 Modèle TCP/IP (4 couches)

Modèle pratique utilisé sur Internet :

| Couche TCP/IP | Équivalent OSI | Protocoles |
|---------------|----------------|------------|
| Application | 5, 6, 7 | HTTP, DNS, SSH, FTP |
| Transport | 4 | TCP, UDP |
| Internet | 3 | IP, ICMP, NAT |
| Accès réseau | 1, 2 | Ethernet, WiFi |

### 9.3 Encapsulation des Données

Quand vous envoyez des données, chaque couche ajoute son en-tête :

```
[Données]
↓ Couche 4
[TCP Header][Données]
↓ Couche 3
[IP Header][TCP Header][Données]
↓ Couche 2
[Ethernet Header][IP Header][TCP Header][Données][Ethernet Trailer]
```

**Exemple avec notre ping** :
```
Application : "ping google.com"
↓
ICMP : Echo Request
↓
IP : Source=192.168.0.2, Dest=142.250.185.46
↓
Ethernet : MAC source → MAC destination
```

### 9.4 Protocoles par Couche dans notre Projet

**Couche 7 (Application)** :
- DNS : Résolution de "google.com"

**Couche 4 (Transport)** :
- Pas utilisée directement (ICMP est couche 3)

**Couche 3 (Réseau)** :
- IP : Adressage 192.168.0.2 → 142.250.x.x
- ICMP : Protocole de ping
- NAT : Traduction d'adresses

**Couche 2 (Liaison)** :
- Ethernet : Communication sur le réseau local
- ARP : Résolution MAC de 192.168.0.1

**Couche 1 (Physique)** :
- Virtuelle dans VirtualBox     

## 10. VirtualBox et Virtualisation Réseau

### 10.1 Modes Réseau VirtualBox

**1. NAT (Network Address Translation)** :
- VM peut accéder à Internet
- VM invisible depuis l'extérieur
- Chaque VM a son propre réseau NAT (10.0.2.0/24)
- IP de la VM : généralement 10.0.2.15
- Gateway : 10.0.2.2

**Utilisation dans le projet** :
- Interface NIC de la box → Accès Internet

**2. Réseau Interne (Internal Network)** :
- VMs peuvent communiquer entre elles
- Isolé de l'hôte et d'Internet
- Nécessite un routeur/gateway pour accéder à Internet
- Nom du réseau : `intnet`, `intnet1`, etc.


## Diagnostics et Dépannage Réseau

### Commandes de Diagnostic

**Tester la connectivité** :
```sh
# Ping local
ping 127.0.0.1        # Tester la pile TCP/IP

# Ping gateway
ping 192.168.0.1      # Tester le réseau local

# Ping DNS
ping 8.8.8.8          # Tester Internet (sans DNS)

# Ping nom de domaine
ping google.com       # Tester DNS + Internet
```



### Problèmes Courants et Solutions

**Pas d'adresse IP** :
```sh
# Vérifier état DHCP
journalctl -u dhclient

# Renouveler le bail
dhclient -r enps03 && dhclient enps03

# Vérifier si interface UP
ip link set enps03 up
```

**Pas de gateway** :
```sh
# Ajouter route par défaut
ip route add default via 192.168.0.1

# Vérifier table de routage
ip route show
```

**DNS ne fonctionne pas** :
```sh
# Vérifier /etc/resolv.conf
cat /etc/resolv.conf

# Ajouter DNS manuellement
echo "nameserver 8.8.8.8" >> /etc/resolv.conf
```

## Quelques liens importants

[Network Configuration](https://wiki.debian.org/NetworkConfiguration#Using_DHCP_to_automatically_configure_the_interface)     
[IPv4 vs IPv6](https://www.geeksforgeeks.org/computer-networks/differences-between-ipv4-and-ipv6/)           
[NIC (Network Interface Card)](https://www.geeksforgeeks.org/computer-networks/nic-full-form/)        
[LAN (Local area network)](https://www.geeksforgeeks.org/computer-networks/lan-full-form/)           
[Computer Network](https://www.geeksforgeeks.org/computer-networks/basics-computer-networking/)     
[Virtual Networking from manual of Oracle VM VirtualBox](https://docs.oracle.com/en/virtualization/virtualbox/6.0/user/networkingdetails.html)