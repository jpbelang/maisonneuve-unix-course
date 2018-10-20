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
 