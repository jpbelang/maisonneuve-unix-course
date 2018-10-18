# Problèmes

yum install tcpdump
tcpdump, curl.

# Securité et cryptographie.

* Sécurité:  les usagers sont le plus gros probleme.
* Sécurité:  tout s'arrête à la sécurité physique. Si quelqu'un a accès au hardware, c'est pratiquement fini.

La cryptographie à la rescousse!!!

## Algorithmes de cryptographie 

* One-time-pad.

* Algorithmes symétriques.

Exemples: Blowfish, Twofish, Serpent, AES (Rijndael),  CAST5, Kuznyechik, RC4, DES, 3DES, Skipjack
SSH: Blowfish, Twofish, 3DES, CAST-128, ARCFOUR

* Algorithmes asymétriques.

Exemples: RSA, Diffie-Hellman, Digital Signature Algorithm, ElGamal, ECDSA, XTR
https://www.digitalocean.com/community/tutorials/understanding-the-ssh-encryption-and-connection-process

* Hashing

* Crypto-systèmes.

Authentification, Signature.

#SSH

* Both parties agree on a large prime number, which will serve as a seed value.
* Both parties agree on an encryption generator (typically AES), which will be used to manipulate the values in a predefined way.
* Independently, each party comes up with another prime number which is kept secret from the other party. 
This number is used as the private key for this interaction (different than the private SSH key used for authentication).
* The generated private key, the encryption generator, and the shared prime number are used to generate a public key 
that is derived from the private key, but which can be shared with the other party.
* Both participants then exchange their generated public keys.
* The receiving entity uses their own private key, the other party's public key, and the original shared prime number 
to compute a shared secret key. Although this is independently computed by each party, using opposite 
private and public keys, it will result in the same shared secret key.
* The shared secret is then used to encrypt all communication that follows.


# Exercice