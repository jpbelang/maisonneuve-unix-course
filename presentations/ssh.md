# ssh
---
# Mécaniques d'encryption

On parlera ici d'Alice (A) et Bob (B), peux partis qui veulent échanger un message sans que Eve (E) ne puisse falsifier
ou decoder le message.

One time pad: Alice et Bob partagent une séquence de nombre aléatoires, souvent dans un livre.  La lettre à la position X de la correspondance
doit être augmentée du nombre à la position X de la séquence.  Une fois terminé, on détruit les nombres utilisés de la séquence.

* La sécurité est parfaite à moins qu'Eve ne mette la main sur une copie de la séquence commune.

* C'est impossible d'échanger la séquence à moins de ne se rencontrer.

* La séquence de codes doit être relativement courte, car le livre doit être portable.
---
# Mécaniques d'encryption

Algorithmes d'encryption à clé symétrique:  Alice et Bob utilisent deux fonctions publiquement connues E(K, message) et D(K, message) ou K est une clé
secrète partagée entre les deux partis.  Si Alice veut envoyer un message a Bob, elle cryptera le message avec la fonction E en utilisant sa clé.
Bob pourra ensuite la décoder avec la fonction D, en utilisant la clé partagée.

* La sécurité dépend maintenant de la longueur de la clé partagée:  une clé de largeur 1, par exemple, demanderait 255 tentatives de dé-cryption 
pour que Eve trouve le message.

* Il n'y a plus de limite de taille.

* Les deux usagers doivent toujours s'échanger les clés sans que Eve ne les voit.  En termes de communication réseau, ce n'est pas pratique.
---
# Mécaniques d'encryption

Algorithmes à clé publique:  Alice et Bob possèdent chacun d'eux deux clé reliés mathématiquement: Kpub (clé publique, que tout 
le monde peut connaître) et Kpriv (clé privée, qui doit demeurer secrète) et deux fonctions E(K, message) et D(K, message).  Pour échanger 
de l'information avec Bob, Alice doit prendre Kpriv de Bob et crypter l'information à envoyer avec E(K, message).  Ensuite Bob utilisera sa 
Kpriv pour dé-crypter le message avec D(K, message).

* Développé par Whitford Diffie et Martin Hellman (Souvent appelé échange Diffie-Hellman).

* Résoud le problème d'echange de clé en public.

* Généralement très coûteux: les largeur de clés sont beaucoup plus grandes que pour les algorithmes a clé symétriques.

* Les méchaniques permettent aussi la signature de message, garantissant que l'originateur du message est bien la bonne personne:
Si Bob signe un message avec sa clé privée, l'univers au complet peut confirmer qu'il est bien l'origine du message en prenant sa clé
publique et vérifiant que le message est déchiffrable.
 
---
# Mécaniques d'encryption

Dans la vraie vie, on utilisera souvent Diffie-Hellman que pour s'échanger une clé symétrique ou pour vérifier l'origine du message.

---
# Pendant ce temps....

Il existe plusieurs raisons de se connecter à un système via une connection réseau:  l'automatisation de tâches, par exemple, est un exemple
flagrant, surtout dans un univers info-nuagique.  On doit souvent se connecter à un parc de machines virtuelles pour les configurer.  Il
est important que la tâche soit simple et sans douleur.  Les choses compliquées amènent des problèmes de sécurité.

Deux problèmes surviennent comme conséquence de cette nécessité.  La sécurite de la communication en tant que telle doit être assurée: on ne veut pas 
qu'un usager malveillant s'installe entre nous et nos systèmes pour _sniffer_ le contenu de la connexion.  

Secundo, il faut garantir notre identité au serveur distant.  Normalement, on fait cela par un système de mot de passe.  Mais automatiser
un processus qui opérera sur 150 systèmes pour ensuite avoir a piocher son mot de passe 150 fois, c'est moyennement intéressant.

Arrive `ssh`, une boîte a outils servant a régler justement tous ces problèmes.  

---
# SSH.

SSH (**S**ecure **SH**ell) est une suite d'outils qui permettent de se connecter de manière sécuritaire avec une authentification forte.  Elle permet:

* De se connecter pour des session interactives (`ssh` sans argument).

* De se connecter à distance pour exécuter une commande (`ssh`).

* De copier des fichiers d'un système à un autre (`scp`).

---
# La session interactive

Il est simple d'ouvrir une session interactive:

```
Biggly:~ jpbelang$ ssh  jpbelang@myhost.com
no such identity: /Users/jpbelang/.ssh/id_dsa: No such file or directory
jpbelang@myhost.com's password: 
Last login: Sat Oct 20 13:58:29 2018 from gateway
[jpbelang@myhost.com ~]$ 
``` 

Ici, sans configuration supplémentaire, on peut se connecter à distance.  Le mot de passe ne peut pas être intercepté sur le réseau,
étant donné que la connexion est encryptée.
---
# La commande lancée

On lance un commande en ajoutant la commande en argument:

```
Biggly:~ jpbelang$ ssh jpbelang@host.com "ls /tmp"
no such identity: /Users/jpbelang/.ssh/id_dsa: No such file or directory
jpbelang@localhost's password: 
systemd-private-9d2d75cf1aa844c1abc22a665714c6e5-chronyd.service-ngp92W
systemd-private-bb351f3efcff4e66943fc10fc7f7a059-chronyd.service-kcrG1G
Biggly:~ jpbelang$ 
```

Le résultat est le listing du répertoire `/tmp` sur le système distant.

---
# Éviter les mots de passe

Dans le cas d'exécution a distance, il est important de ne pas avoir à pitonner son mot de passe.  Mais, en même temps, 
on doit s'assurer que l'appelant est bel et bien la bonne personne.  Avec SSH, la solution est une paire de clés asymétriques.  

Primo, il faut créer la paire de clés sur notre système local:

```
[jpbelang@localhost ~]$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/jpbelang/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/jpbelang/.ssh/id_rsa.
Your public key has been saved in /home/jpbelang/.ssh/id_rsa.
The key fingerprint is:
SHA256:3vQms/rknZCDv4RJrDk+dd2FEQJkkLj5JHOM3iqWzXs jpbelang@localhost.localdomain
The key's randomart image is:
+---[RSA 2048]----+
|       ..++.. .. |
|      . ..   ..  |
|       =       o |
|      *.+     . .|
|     . BS .. . . |
|      .=+*.o. .  |
|     ++o=.X o    |
|    +.=.E= O .   |
|   . .o+.o*.o    |
+----[SHA256]-----+
[jpbelang@localhost ~]$ 

```

À remarquer:  on n'a pas mis de mot de passe sur la clé.  C'est pour faire simplement.  On fera un exemple avec un mot de passe plus tard.

----
#  ssh-copy-id

On copie ensuite la cle publique sur le système distant.
```
Biggly:~ jpbelang$ ssh-copy-id -f -i ~/.ssh/id_rsa jpbelang@host.com
/opt/local/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/Users/jpbelang/.ssh/id_rsa.pub"
jpbelang@localhost's password: 

Number of key(s) added:        1

Now try logging into the machine, with:   "ssh 'host.com'"
and check to make sure that only the key(s) you wanted were added.
```

On se connecte ensuite sans mot de passe:
```
Biggly:~ jpbelang$ ssh jpbelang@host.com 
Last login: Sat Oct 20 16:22:31 2018 from gateway
[jpbelang@localhost ~]$ 
```

Pas de mot de passe!  L'usager est validé par un *challenge* cryptographique lancé par le serveur. 
**Il est important de se rapeller que les clé privées et publiques configurées précédemment ne sont pas utilisées pour l'encryption de la communication**
Elles ne sont utilisées que pour l'authentification. 
---
# Les tunnels

Il est possible d'utiliser SSH comme tunnel sécurisé pour d'autres protocoles. Par exemple, des protocoles comme VNC (un 
protocole de partage d'écran), IMAP (un protocole de gestion de boîtes de courrier électronique).  

Supposons qu'un usager est sur un système local mais doit accéder un serveur IMAP qui est derrière un *firewall* accessible via SSH.  Supposons aussi qu'il veuille 
se connecter en utilisant un programme de courrier normal. Il pourrait, avec SSH, se connecter à un port local qu'SSH amènerait jusqu'à un port et serveur distant.

```
+---------------+       SSH         +--------------------+    IMAP   +-----------------------+
|     Local     |------------------>|      Firewall      |---------->|     Serveur IMAP      |
+---------------+                   +--------------------+           +-----------------------+
```   

La commande pour faire cela:

```
ssh -N -L 2143:imap-server.com:143 jpbelang@firewall.com
```

Cette commande dit à SSH "Lance une connexion non-interactive (-N) qui ouvre un port local (-L) 2143 pour lequel, un fois rendu au serveur SSH, 
tout le traffic sera routé au serveur IMAP sur le port normla IMAP (143)".  L'usager n'a qu'à pointer son client email au serveur local, port 2143
et il aura acces à son courrier.  Toute la connexion SSH est sécurisée. 
