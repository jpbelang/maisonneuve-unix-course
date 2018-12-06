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

`yum install samba samba-client cifs-utils`

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

Si tu t'attaches à ce "répertoire" avec ton noum d'usager, il partagera ton répertoire personnel.

------
# Côté serveur (sécurite)

On peut controller l'accès aux shares.
```
[public]
	path = /usr/local
	read only = yes
	guest ok = yes
```

Ou par usager.  Si on n'utlilise pas de PDC, on doit ajouter le mot de passe dans une bd locale (on ne peut pas effectivement 
utiliser son mot de passe Unix dans un réseau moderne:

```
smbpasswd -a jpbelang
smbpasswd -e jpbelang
```

```
[public]
	path = /usr/local
	read only = yes
	guest ok = no
```
------

# Côté client

La manière de tester la plus simple est d'utiliser `smbclient`.  C'est une commande qui permet de se connecter sur des cibles 
smb.


```
[jpbelang@localhost ~]$ smbclient '\\localhost\public'
Enter SAMBA\jpbelang's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Sep 16 13:31:55 2018
  ..                                  D        0  Sun Sep 16 13:31:56 2018
  bin                                 D        0  Wed Apr 11 00:59:55 2018
  etc                                 D        0  Wed Apr 11 00:59:55 2018
  games                               D        0  Wed Apr 11 00:59:55 2018
  include                             D        0  Wed Apr 11 00:59:55 2018
  lib                                 D        0  Wed Apr 11 00:59:55 2018
  lib64                               D        0  Wed Apr 11 00:59:55 2018
  libexec                             D        0  Wed Apr 11 00:59:55 2018
  sbin                                D        0  Wed Apr 11 00:59:55 2018
  share                               D        0  Sun Sep 16 13:31:55 2018
  src                                 D        0  Wed Apr 11 00:59:55 2018

		17811456 blocks of size 1024. 15708640 blocks available
smb: \> 
```
-----
# Côté client

On peut aussi utiliser la commande `mount`:

```
[root@localhost jpbelang]# mount -t cifs -o username=jpbelang //localhost/public /media
Password for jpbelang@//localhost/public:  *******
```

On peut passer le mot de passe sur la ligne de commande (une très mauvaise idée), ou on peut creer un fichier `credentials`
que l'on peu placer ou l'on veut et qui contient le nom et mot de passe à utiliser pour la connexion:

```
username=jpbelang
password=secret
```

```
[root@localhost jpbelang]# mount -t cifs -o credentials=/wherever/credentials //localhost/public /media
```

------
# Courte parenthèse sur l'automonteur

Il peut devenir complexe et souvent inefficace de monter tous les répertoires réseaux sur un serveru au ca ou un usager
ait besoin de s'en servir.

*  Ça met de la complexité du coté client.
*  Ça met de la charge inutile sur le serveur.
*  Ça ne permet pas aux clients de changer de serveur si un de ceux ci meurt.

L'automonteur est un peu une réponse à tous ces problèmes.

------

# auto.master

```
/misc   /etc/auto.misc  # ici, on pourrai mettre des options.
```

----
# auto.misc

```
software        -fstype=nfs4,rw,soft,intr       192.168.12.5:/software
data            -fstype=nfs4,rw,soft,intr       192.168.12.5:/data
scripts         -fstype=nfs4,rw,soft,intr       192.168.12.5:/scripts \
                                                192.168.12.6:/scripts
```

