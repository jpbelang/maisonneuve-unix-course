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
