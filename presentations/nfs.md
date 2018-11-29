# NFS

Network file system
----
# Qu'est-ce qu'NFS ?

NFS, c'est le grand-père des systèmes de fichier réseau

* Il est assez simple à configurer.

* Il est assez puissant, surtout lorsque combiné avec l'auto-monteur disponible avec la plupart des systèmes Unix.

* C'est un trou de sécurité assez massif lorsque exposé à l'Internet.  Ne jamais présenter ce type de service directement 
sur un réseau public.

-----
# NFS repose sur RPC

C'est souvent d'ici que partent les problèmes de sécurité.

* NFS repose sur au moins 3 ports et peut être transmis sur UDP et TCP.  Ça complique les règles de firewall.

* NFS repose sur l'utilisation du `portmapper` qui a toujours été une foire a problèmes de sécurité.  Il ne devrait jamais
être visible sur un réseau public.

* En général, les protocoles de partage de fichiers ne sont pas faits pour circuler sur l'Internet.  
En général, on utilisera HTTP/HTTPS.

----
# Configuration du serveur

Le package d'intérêt général `nfs-utils`.

La configuration du serveur se fait dans `/etc/exports`.

```
/home *(rw)
/software 192.168.*(ro)
/data *.local.company.com(rw, no_root_squash, no_all_squash), *.remote.company.com(ro)
```

Ensuite, y'a l'option `sync` ou `async`, qui permet des optimisations de performance en concédant de la sécurite de données.

That's it.  Y'a rien d'autre.
  
----
# Configuration du client

Le client utilise la fonctionnalité apportée par `mount`.
 
`mount -o ro 10.3.1.13:/home /media`

Les options importantes sont:

`soft` vs. `hard`:  `hard` indique que le client ré-essayera ad infinitum, `soft` indique un nombre de tentatives limitées. 
`retrans`:  nombre de retransmissions essayées avant de déclarer un échec.
`timeo`:  en dixièmes de secondes, la période de temps que le client attend une réponse du serveur avant de déterminer un échec.
