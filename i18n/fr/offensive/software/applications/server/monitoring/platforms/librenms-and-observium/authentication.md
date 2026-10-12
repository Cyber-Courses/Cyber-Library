---
title: "Authentification : identifiants LibreNMS/Observium par défaut et faibles"
description: "Les interfaces web de LibreNMS et d'Observium s'authentifient avec un mot de passe et sont exposées aux identifiants par défaut et faibles, y compris les comptes admin créés à l'installation et les mots de passe réutilisés. Une connexion web donne accès à la configuration des équipements détenant les identifiants stockés et aux fonctionnalités authentifiées derrière les vulnérabilités d'injection de commandes."
keywords:
  - librenms login
  - observium
  - default credentials
  - weak password
  - admin
---

# Authentification

Les deux plateformes protègent leur interface web avec un mot de passe, et l'accès est généralement obtenu via des identifiants par défaut ou faibles. LibreNMS crée un compte admin à l'installation dont le mot de passe est choisi par l'opérateur et souvent faible ou réutilisé ; Observium est livré avec un compte par défaut documenté (`admin`/`admin` dans sa configuration) qui est fréquemment inchangé. Aucun n'applique une politique robuste par défaut sur les versions plus anciennes. Une connexion web donne la vue de monitoring et la configuration des équipements/sources de données où sont stockés les chaînes SNMP et les identifiants, et atteint les fonctionnalités authentifiées qu'exploitent les vulnérabilités d'injection de commandes, de sorte que les identifiants sont l'étape d'accès avant la récolte comme avant le RCE.

```bash
# Observium default
curl -sk -c cj https://<target>/ --data 'username=admin&password=admin'
# LibreNMS form login (CSRF token in the login page)
curl -sk -c cj https://<target>/login                       # grab _token
curl -sk -b cj -c cj https://<target>/login --data '_token=<t>&username=admin&password=<pw>'
# spray weak/reused passwords against the identified product
```

## Notes d'exploitation

- Essayez les comptes par défaut du produit (`admin`/`admin` sur Observium ; l'admin d'installation sur LibreNMS) et les mots de passe faibles/réutilisés ; l'absence de politique robuste sur les versions plus anciennes rend le spraying efficace.
- L'accès admin débloque la [récolte d'identifiants](credential-harvesting.md) (identifiants d'équipements stockés) et les voies d'[injection de commandes](command-injection.md) authentifiées vers le RCE.
- LibreNMS émet aussi des jetons d'API ; un jeton trouvé s'authentifie directement auprès de l'API, contournant le formulaire.
- Une connexion est la précondition commune des deux suites ; les comptes non-admin ont une portée limitée.

## Références

- [LibreNMS: authentication](https://docs.librenms.org/Extensions/Authentication/)
- [Observium documentation](https://docs.observium.org/)
