#HTTP

HTTP, le protocol to end all protocols.

----

# Installation
 `yum install httpd`
 
----
 #Le protocole.

C'est un protocole très simple qui permet le transfert de données.

* Fonctionne normalement sur le port 80 (non-sécurisé) ou bien 443 (HTTPS, sécurisé)
* Le protocole est en texte lisible.
* Le protocole est un protocole requête-réponse sans état.
* Les messages sont divisés en trois parties:  la requête, les entêtes et le corps.
* Il y a eu deux grandes versions déployées sur l'internet: 1.0 et 1.1.
* La version 2.0 est en déploiement/développement présentement et est une refonte assez complète du protole.
----
# Une requête

```
GET /docs/index.html HTTP/1.1
Host: www.nowhere123.com
Accept: image/gif, image/jpeg, */*
Accept-Language: en-us
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)
<vide>
```

----
#Une réponse

```
HTTP/1.1 200 OK
Date: Sun, 18 Oct 2009 08:56:53 GMT
Server: Apache/2.2.14 (Win32)
Last-Modified: Sat, 20 Nov 2004 07:16:26 GMT
ETag: "10000000565a5-2c-3e94b66c2e680"
Accept-Ranges: bytes
Content-Length: 44
Connection: close
Content-Type: text/html
X-Pad: avoid browser bug
  
<html><body><h1>It works!</h1></body></html>
```
----
# Rapidement, les verbes et les réponses et obligations

`GET`, `PUT`, `POST`, `DELETE`, `PATCH`, `HEAD`, `OPTIONS`, `CONNECT`

`1xx`: Réponse provisionnelle.
`2xx`: Succès.
`3xx`: Redirection.
`4xx`: Requête invalide.
`5xx`: Erreur du serveur.

Les réponses doivent contenir une entête `Content-type`.
Les requêtes HTTP 1.0 ne demandent rien comme entête.
Les requêtes HTTP 1.1 demandent le header `Host`.
----
# C'est un protocole sans état

Une requête, une réponse.  That's it, that's all.  L'état doit être maintenu ou bien par le client, 
ou bien par l'application dans le serveur.

Cette maintenance est souvent faite par des cookies HTTP.
---

# Configuration (base)

La configuration générale se fait dans les fichiers dans `/etc/httpd/conf/httpd.conf`
Premièrement:  ou écoute le serveur ?
```
Listen *:80
Listen 12.34.23.45:81
```

Deuxièmement on fait la configuration particulière à notre application (ou "instance"), généralement dans `/etc/httpd/conf.d/myhost.conf`
```
<VirtualHost *:80>
  ServerAdmin webmaster@bind.com
  DocumentRoot "/var/www/bind"
  ServerName bind.com
  ErrorLog "logs/host.bind.com-error_log"
  TransferLog "logs/host.bind.com-access_log"
  Options Indexes ExecCGI
</VirtualHost>
```

Ici, tout le traffic arrivant du port 80, ayant comme cible `bind.com` sera routée a cette configuration.
On peut configurer les permissions par répertoire dans une clause `VirtualHost`.
----
# HTTPS
La configuration est très simple:
```
Listen *:443

<VirtualHost *:443>
   SSLEngine On
   SSLCertificateFile "/etc/ssl/certs/apache-selfsigned.crt"
   SSLCertificateKeyFile "/etc/ssl/private/apache-selfsigned.key"
   ServerAdmin info@example.com
   ServerName bind.com
   DocumentRoot "/var/www/bind"
   ErrorLog "logs/host.bind.com-error_log"
   TransferLog "logs/host.bind.com-access_log"
</VirtualHost>
```
----
# Parenthèse de génération de certificat.....

`openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/selfsigned.key -out /etc/ssl/certs/selfsigned.crt`

Et les questions:
```
Country Name (2 letter code) [XX]:CA
State or Province Name (full name) []:QC
Locality Name (eg, city) [Default City]:Montreal
Organization Name (eg, company) [Default Company Ltd]:Whatevs
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:*
Email Address []:
```

----
# Proxying

Un proxy est un "middleman" qui intercepte les requêtes et les retransmets.  Il existe deux sortes de proxy, d'un point de vue de leur utilisation:
le proxy "normal" et le proxy "reverse".  

* Le proxy normal est utilisé pour cacher l'information demandée à l'interne d'une compagnie.  Apache peut faire cette operation, mais il existe des implémentations plus populaires,
  telles que [Squid](http://www.squid-cache.org/).

* Le proxy "reverse":  c'est un proxy qui se trouve proche d'une application qui permet de faire du caching et/ou du load balancing.
-----
# Configuration d'un proxy normal

Dans le fichier de config général ou bien dans le fichier particulier:
```
Listen 10.10.10.1:8080
```

et le virtual host:

```
 
<VirtualHost 10.10.10.1:8080>
  ProxyRequests On
  SSLProxyEngine On
</VirtualHost
```   

----
# Reverse proxy (simple)

```
<VirtualHost *:81>
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:80/
    ErrorLog "logs/host.proxyerror_log"
    TransferLog "logs/host.proxy-access_log"
</VirtualHost>
```

----
# Load balancer

```
<VirtualHost *:80>
    <Proxy balancer://cluster>
        BalancerMember http://1.2.3.110:80/test
        BalancerMember http://2.3.4.120:80/test
    </Proxy>

    ProxyPreserveHost On
    ProxyPass / balancer://cluster/
    ProxyPassReverse / balancer://cluster/
</VirtualHost>
```
----
# Grosse configuration!

```
+---------------+       HTTPS       +--------------------+    HTTP   +-----------------------+
|    Browser    |------------------>|      Proxy         |---------->|  Documentation BIND   |
+---------------+                   +--------------------+           +-----------------------+

Dans le browser on doit ecrire:

    https://nouvellecompagnie.inet:8080/Bv9ARM.ch01.html

    Et on doit afficher la page Bv9ARM.ch01.html.
    
    PS:  oui, ca va prendre un setup DNS.
    PPS:  oui, vous pouvez le faire à deux, du moment que les deux co-equipiers soient présents.
``` 

