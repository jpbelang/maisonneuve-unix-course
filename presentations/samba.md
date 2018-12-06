# Samba

----

# SMB

C'est un protocole de partage de fichiers développé par Microsoft.  Le protocole est propriétaire.

On en est à la version 3.1.1.

Le protocole:
* Exécute, par défaut, sur les ports TCP 445 (direct) ou bien 137 ou 139 (NetBIOS).
* Supporte l'encryption.

Les implémentations originales n'opéraient que sur Windows (et sa descendance).  Depuis la 
version 2.0, la spécification de SMB, toujours propriété de Microsoft, est publiée.

-----
# Samba

Le logiciel Samba est une implémentation open-source du protocole développé en mode "clean room" 
par Andrew Tridgell.

Samba supporte:
* NetBIOS
* WINS
* NT Domain Login
* L'impression
* Serveur Active Directory

Il peut donc remplacer beaucoup de logiciels Microsoft et permet une integration simple de systèmes Unix avec ceux de
Microsoft.  

On en est à la version 4.

------
# Installation

`yum install samba samba-client`

-----
# Côté serveur

La configuration de Samba se rapproche de celle de NFS:  On définit un répertoire que l'on veut partager et on lui assigne des 
paramètres de partage. La syntaxe ressemble à:

```
[software]
       comment = Public Software
       path = /software
       public = yes
       writable = no
       printable = no
```

En gros, chaque "partage" commence avec une section entre crochets et est suivi d'une liste de paramètres.

-----
# Côté serveur

Les partages dépassent les fichiers:  on peut aussi partager les imprimantes.
```
[printers]
        comment = Imprimantes
        path = /var/spool/samba
        browseable = no
        guest ok = no
        writable = no
        printable = yes
```

Les imprimantes disponibles sous Unix seront partagées aux usagers Windows.

-----
# Côté serveur

Les répertoires usagers sont l'objet d'une configuration particulière:

```
[homes]
        comment = Home Directories
        browseable = no
        writable = yes
;       valid users = %S
;       valid users = MYDOMAIN\%S

```