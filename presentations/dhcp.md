# DHCP

Ze Dynamic Host Configuration Protocol

---
# L'assignation d'adresses IP

Un des grands problèmes de TCP/IP est l'assignation des adresses:  tout conflit d'adresse peut mener a des gros problèmes 
dans le réseau.

On a donc deux choix. Le premier est de configurer chaque système avant de le livrer.  Il y a quelques problèmes avec une 
approche de ce type.

* Un changement de plan d'adressage forcerait la reconfiguration de tous les ordinateurs sur le plancher.

* Dans un contexte hautement mobile, le nombre d'adresses peut rapidement devenir difficile a gérer, surtout si beaucoup 
d'adresses sont éphémères.

Assigner les adresses sur les très grands parcs d'ordinateurs peut rapidement devenir un mal de tête.

---
# DHCP (et feu BOOTP) à la rescousse.

DHCP permet à la configuration d'un réseau d'être externe.  DHCP permet:

* D'assigner des adresses, gateway et netmasks dynamiquement.

* D'assigner les serveurs DNS, ....

* D'assigner toutes sortes de paramètres qu'un système devrait avoir au moment de son lancement.

---
# DHCP, le protocole.

Le protocole DHCP fonctionne sous UDP, souvent en mode broadcast.  Au lancement de l'ordinateur:
 
* Le client lance un broadcast DHCPDISCOVER.  Dans cette requête, il y a assez d'information pour que le serveur DHCP
puisse faire une offre DHCP, incluant l'information que le client requiert au serveur. 

* Le serveur répond avec un DHCPOFFER, pour offrir une adresse (et plusieurs autres configurations).

* Le client répondra ensuite à l'offre au serveur de son choix.

* Le serveur répond finalement avec un DHCPACK.  Après cette requête, la transaction est finalisée. 
---
# Le protocole

Les messages envoyés en mode broadcast ne sont pas routés au travers des segments.  On devra ou bien:

* Mettre un serveur DHCP sur chaque segment.  Cette solution n'est pas pratique côté gestion d'adresses.

* Installer un agent de relais DHCP, qui transporte les broadcasts jusqu'au serveur DHCP principal, en utilisant des messages UDP unicast.
De cette manière, on peut garder l'information centralisée.

----
# Le bail

L'information attribuée a un ordinateur a une durée de vie.  Spécifiquement, l'adresse IP du système est fixée
pour une période.  Le serveur respectera ce bail (lease) et ne ré-attribuera pas l'adresse.  Il est impossible que le serveur
ne DHCP révoque le bail:  il n'y a en fait aucune garantie que les systèmes client soient connectés au moment ou le serveur 
demanderait la révocation. 

La durée de ce bail est configurable et dépend beaucoup de l'usage du réseau. Si le bail est trop court, il est fort possible
qu'un usager vive des interruptions de service s'il n'est pas capable de rejoindre le serveur DHCP. S'il est trop long, il 
est possible de manquer d'adresses IP et que, conséquemment, il soit impossible de se connecter au réseau. 

Quelques exemples:  dans un milieu corporatif, avec un parc d'ordinateurs bien défini, on peut étirer la longueur du bail 
jusqu'à une semaine.  Un système où on admet les cellulaires des usagers à se connecter, on peut limiter leur bail à une journée.

---
# Renouvellement

Périodiquement, les clients doivent renouveler leur adresse.  En général, au moment où le bail arrive a 50% de son temps, DHCP tentera de renouveler
celui-ci en contactant directement le serveur duquel il l'a obtenu avec un DHCPREQUEST. Le serveur peut répondre DHCPACK (on continue le bail, 
ou voici une nouvelle adresse), ou DHCPNAK (le bail est terminé).
 
S'il ne réussi pas à contacter le serveur, a 87,5% du temps, il retentera avec un DHCPDISCOVER de trouver un nouveau serveur. 

---
# Reboot

Si un système est relancé main possède un bail actif, le système tentera de renouveler son bail avec le serveur.  Si le serveur 
répond, le système réagit de manière appropriée.  Sinon, le client tente lnce un ICMP PING ver le gateway par défaut présentement
configuré, et si le gateway répond, le client considère son bail comme étant toujours valide et continue de l'utiliser.  
Sinon, il tombe en mode APIPA. 

# APIPA.

Si rien ne fonctionne, plusieurs systèmes utiliseront une configuration **A**utomatic **P**ivate **IP** **A**ddressing. Essentiellement,
le système s'auto-assigne une adresse dans la plage 169.254.0.0.  Avant de s'assigne un de ces adresses, il lance une requête ARP
utilisant cette adresse pour s'assurer que personne n'y réponde.

Ces adresses ne sont pas routables mais permettent a un système non configuré d'être accessible par réseau.


---
# Le serveur DHCP sous CentOS 7

 
Le serveur DHCP de CentOS 7 est dhcpd.  Il n'est pas installé par défaut.

```
sudo yum install dhcpd 
```

La configuration vit dans `/etc/dhcp/dhcp.conf`

--- 
# La configuration
```
subnet 10.3.1.0 netmask 255.255.255.0 {
     range 10.3.1.100 10.3.1.200;
     option domain-name-servers 8.8.8.8, 8.8.4.4;
     option domain-name "happy.local";
     option routers 10.3.1.1;
     option broadcast-address 10.3.1.255;
     default-lease-time 600;
     max-lease-time 7200;
}
```
---
# More configuration

```
shared-network others {
	subnet 10.4.1.0 netmask 255.255.255.0 {
		range 10.4.1.100 10.4.1.200;
		option domain-name-servers 8.8.8.8, 8.8.4.4;
 		option domain-name "happy.local";
 		option routers 10.4.1.1;
 		option broadcast-address 10.4.1.255;
 		default-lease-time 600;
 		max-lease-time 7200;
	}
}
```