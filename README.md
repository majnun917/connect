<h1 align="center">Connect</h1>

Pour ce projet on doit gerer la connectivite entre 3 VM.        
On a le controle sur une seule machine(``machine 2``) et cette machine va se connecter grace grace a la premiere machine(``box``), qui aura le role de route. Cette box va avoir acces à internet grace au reseau 
[NAT](https://en.wikipedia.org/wiki/Network_address_translation).
Et pour que la connection passe les machines ont besoin d'adresse [IP](https://en.wikipedia.org/wiki/IP_address).
Les adresses IP can peuvent etre generees d'une maniere ``static`` ou demander à un [serveur DHCP](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol) d'assigner à la machine une ``addresse dynamique``.

L'image suivante illustre la configuration des machines:
<img src="config.png" width="60%">

On commence par enumerer les etapes de la configuration des VM, ensuite quelques commandes qui permettent de gerer les conflits car si vous remarque sur l'image on a les addresses IP sur les 2VM clients et enfin expliquer quelques concepts utilises dans ce projet.

## Les etapes de la configuration       
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
Et s'il y'a des conflits dus aux adresses IP identiques. On peut changer l'addresse IP d'une par deux methodes:
- Methode Static: on edite manuellement le fichier  /etc/network/interfaces:
```sh
nano /etc/network/interfaces
```     
Et modifie par:     

``` sh 
Et on specificie l'addresse IP:
    auto enps03(interface name)
    iface enps03 inet static
    address 192.0.2.3 // Ici on a changé .2 par .3
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


(https://www.geeksforgeeks.org/computer-networks/differences-between-ipv4-and-ipv6/)