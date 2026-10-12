---
title: "WAF : contourner les règles d'une web ACL pour atteindre l'application"
order: 10
description: "Contourner les règles d'AWS WAF via l'encodage, les limites de taille et les lacunes de règles pour atteindre l'application protégée."
keywords:
  - WAF
  - web ACL
  - rule
  - bypass
  - encoding
---

# WAF

AWS WAF applique une web ACL de règles gérées et personnalisées à CloudFront, API Gateway ou un ALB. Lire la web ACL révèle quelles règles s'exécutent ainsi que leurs limites de taille de requête et de correspondance, et l'attaque consiste à façonner une charge utile qui se glisse dans une lacune : des corps surdimensionnés que WAF cesse d'inspecter, des encodages que la règle ne normalise pas, ou un chemin de requête qui atteint l'origine sans jamais passer par l'ACL.

## Lire la web ACL

```bash
aws wafv2 list-web-acls --scope REGIONAL   # or CLOUDFRONT
aws wafv2 get-web-acl --name <name> --scope REGIONAL --id <id> \
  --query "WebACL.Rules[].[Name,Statement,Action]"
```

## Leviers de contournement

```
# oversized body: WAF inspects only the first N KB, so push the payload past the limit
# encoding/normalization gaps the managed rule misses
id=1%2f%2a%2a%2funion%2f%2a%2a%2fselect   # comment/whitespace obfuscation
# header or method the rule does not cover (e.g. only GET/POST inspected)
```

## Notes d'exploitation

- Une protection WAF appliquée uniquement au niveau de CloudFront est inutile si l'[origine](cloudfront.md) accepte les requêtes directes : l'adresse de l'origine est le contournement le plus propre.
- La limite de taille d'inspection du corps est une frontière stricte : le contenu au-delà n'est pas filtré, ce qui convient aux grandes charges utiles d'injection ou d'upload.
- Les règles basées sur le débit (rate-based) et la réputation d'IP sont contournées avec des adresses sources distribuées : inspectez la web ACL pour repérer les règles basées sur la réputation.

## Outils

- **AWS CLI** (`wafv2 get-web-acl`) : divulgation des règles là où vous disposez d'un accès en lecture.
- **Burp Suite** / **ffuf** : itérer les encodages et les tailles contre l'ACL en direct.
- **nuclei** : identifier l'empreinte du WAF et les contournements connus.

## Références

- [HackTricks : WAF bypass](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/waf-bypass.html)
- [AWS : règles de web ACL WAF](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rules.html)
- [Hacking the Cloud : techniques offensives AWS](https://hackingthe.cloud/)
- [CloudFox : trouver les origines exposées derrière l'edge](https://github.com/BishopFox/cloudfox)
