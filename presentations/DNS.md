# DNS 

---

# Rapide historique

Feu, jadis (des débuts d'ARPANET à 1985), les noms des systèmes sur l'Internet étaient géres avec un fichier central qui contenait le nom et l'adresse de 
tous les systèmes sur le réseau.  C'était en fait géré par Jon Postel, un vraie personne en fait très connue pour ses contributions
au monde de la résautique.

Rapidement, on dût trouver une solution pour répondre à la question d'augmentation de taille de l'Internet.  Cette solution 
est le DNS, une solution distribuée de contrôle de noms.

À la base, le DNS mappe des noms vers des adresses, des adresses vers des noms et la délégation de zones de contrôle..  
Se sont greffées à ces fonctions de méthodes pour trouver des serveurs de courrier, et des manières de gérer des services génériques
qui sont utilisés pout SIP et XMPP.

---
# Bases du DNS

Le DNS divise son espace de nom en zones d'authorité séparées, que l'on appelle des domaines.  Chaque domaine est déleguée d'une zone (jusqu'aux serveurs racine) 
et chacune d'entres elles peut ensuite déléguer à d'autres zones.  

Le DNS est donc un arbre de domaine de profondeur théoriquement arbitraire.  En fait, le plus long nom de domaine permis 
est de 253 caractères incluant les séparateurs, et chaque domaine est limité à 63 caractères.

---
#  Noms valides

Originalement, le DNS n'utilisait que les caractères ASCII suivants:  des lettres, des chiffres et le caractère "-".  Maintenant, par contre,
les noms de domaines acceptent des caractères Unicode, transformés vers ASCII par un encodage nommé "punycode".

---
# Installation

`yum install bind`

`yum install bind-utils`

Pour les besoins du cours, on debarque SELinux.

`sudo setenforce 0`, ou bien on edite /etc/selinux/config.

---
# Configuration de base
```
options {
        listen-on port 53 { 127.0.0.1; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        allow-query     { localhost; };

        recursion yes;
        dnssec-enable yes;
        dnssec-validation yes;

        /* Path to ISC DLV key */
        bindkeys-file "/etc/named.iscdlv.key";

        managed-keys-directory "/var/named/dynamic";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";
        forwarders {
                206.167.118.39;
                206.167.40.14;
        };
};

```
---
# Fichier de zones named.conf
```
zone "compagnie.com" {
    type master;
    file "/etc/named/zones/db.compagnie.com"; # zone file path
};

zone "0.10.in-addr.arpa" {
    type master;
    file "/etc/named/zones/db.10.0";  # 10.128.0.0/16 subnet
    };    
```
---
# Définition d'une zone
```
$TTL    604800
@       IN      SOA     compagnie.com admin.compagnie.com. (
             2018122701 ; Serial
             604800     ; Refresh
              86400     ; Retry
            2419200     ; Expire
             604800 )   ; Negative Cache TTL

                  IN      NS     ns1.compagnie.com.
compagnie.com.    IN      NS     ns2.compagnie.com.
                  IN      MX     mail.compagnie.com.
     
; name servers - A records
ns1.compagnie.com.          IN      A       10.0.2.4
ns2.compagnie.com.          IN      A       10.0.2.5

serveur-important.compagnie.com.        IN      A      10.0.2.10
mail.compagnie.com.                     IN      A      10.0.2.11
courrier.compagnie.com.                 IN      CNAME  mail.compagnie.com.
```
---
#  Le "reverse DNS" 
```
$TTL    604800
@       IN      SOA   compagnie.com. admin.compagnie.com. (
                     2018122702         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
; name servers
      IN      NS      ns1.compagnie.com.
      IN      NS      ns2.compagnie.com.

; PTR Records
4.2   IN      PTR     serveur-important.compagnie.com.   
5.2   IN      PTR     mail.compagnie.com.   
```

---
# Délégation d'une zone
```
$TTL    604800
@       IN      SOA   compagnie.com. admin.compagnie.com. (
                     2018122702         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

montreal    IN NS  montreal-ns1.montreal.compagnie.com.
montreal    IN NS  montreal-ns1.montreal.compagnie.com.

montreal-ns1.montreal.compagnie.com. IN A 10.0.3.5
montreal-ns1.montreal.compagnie.com. IN A 10.0.3.6
```
---
# Plusieurs adresses
```
www.compagnie.com.      IN      A       10.0.2.17
                        IN      A       10.0.2.18
                        IN      A       10.2.0.19
```

---
# dig (Domain Information Groper)

C'est l'outil de prédilection pour debugger une configuration DNS.

`dig @localhost www.compagnie.com`

`dig @localhost montreal.compagnie.com NS`

`dig @localhost montreal.compagnie.com AXFR`

---
# Zones secondaires

Sur le master...
```
zone "compagnie.com" IN {
type master;
notify yes;
also-notify { 10.2.0.112; };
};
```

Sur le slave:
```
zone "compagnie.com" IN {
type slave;
masters { 192.168.12.8; };
file "slaves/db.compagnie.com";
};
```

---

# Dynamic DNS

Ça prend une clé!

```
dnssec-keygen -r /dev/urandom -a HMAC-SHA512 -b 512 -n HOST sb.montreal.compagnie.com.
```

Le serveur doit connaitre la cle!

```
key "sb.montreal.compagnie.com." {
    algorithm HMAC-SHA512;
    secret "B7RbiaPtDQM6s+Nj++QgVWMo3UD5/YydbxTeu5Jf+FtAkkn41RCAGEaI 5VNvamU3cjFn5hmNl3IRF6S3qzlVNg==";
};
```

On doit dire au serveur d'accepter la cle!

```
zone "montreal.compagnie.com" IN {
        type master;
        file "montreal.zone.db";
        allow-update {
                key sb.montreal.compagnie.com.;
        };
};
```
---

# Test d'ajout d'adresse

On fait un fichier d'ajout `addressfile`
```
update add hello2.montreal.compagnie.com. 86400 A 10.2.0.20
send
``` 

On lance la commande:

`nsupdate -k Ksb.montreal.compagnie.com.+165+05437.key addressfile`

Tadam.
