---
title: "Authentification : l'admin par défaut et les identifiants Cacti faibles"
order: 1
description: "Cacti s'installe avec un compte admin par défaut (admin/admin), et bien que les versions plus récentes forcent un changement de mot de passe à la première connexion, de nombreuses instances conservent des mots de passe faibles ou devinables. Une connexion web donne accès aux sources de données et aux paramètres qui détiennent les identifiants d'équipements et aux voies de vulnérabilité authentifiées qui atteignent l'exécution de code."
keywords:
  - cacti admin
  - default credentials
  - weak password
  - first login
  - cacti
---

# Authentification

L'administrateur par défaut de Cacti est `admin` avec le mot de passe `admin`. Les versions plus récentes invitent à le changer à la première connexion, mais les installations qui ont été mises à niveau, scriptées, ou simplement ignorées conservent fréquemment `admin`/`admin` ou un remplacement faible, et Cacti n'a historiquement appliqué aucune politique de mot de passe robuste. Une connexion web en tant qu'admin donne accès à la configuration des sources de données et des équipements, où sont stockés les chaînes SNMP et les identifiants que Cacti utilise pour interroger, et aux fonctionnalités d'injection de commandes authentifiée et de configuration que plusieurs de ses vulnérabilités exploitent. Les identifiants par défaut ou faibles sont donc l'étape d'accès courante avant la récolte d'identifiants ou une chaîne de RCE authentifié.

```bash
# default admin login (form posts to index.php with a CSRF token)
curl -sk -c cj https://<target>/cacti/index.php | grep -oE 'csrfMagicToken="[^"]+"'   # grab token
curl -sk -b cj -c cj https://<target>/cacti/index.php \
  --data 'action=login&login_username=admin&login_password=admin&__csrf_magic=<token>'
```

## Notes d'exploitation

- Essayez `admin`/`admin` en premier ; malgré l'invite de première connexion, il persiste sur de nombreuses instances, et les remplacements faibles sont courants.
- La connexion de Cacti utilise un jeton CSRF (`__csrf_magic`) qui doit être récupéré et soumis ; les connexions scriptées le récupèrent d'abord depuis la page de connexion.
- L'accès admin débloque la [récolte d'identifiants](credential-harvesting.md) (les identifiants SNMP/d'équipements stockés) et les portions authentifiées des [chaînes d'exploitation](known-exploits.md) ; certains RCE Cacti sont non authentifiés et ne nécessitent aucune connexion.
- Les mots de passe faibles/réutilisés et l'absence de verrouillage sur les versions plus anciennes rendent le spraying viable.

## Références

- [Cacti: user management](https://docs.cacti.net/)
- [HackTricks: Cacti](https://book.hacktricks.xyz/)
