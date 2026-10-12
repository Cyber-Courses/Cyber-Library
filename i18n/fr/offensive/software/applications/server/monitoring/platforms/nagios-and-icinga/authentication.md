---
title: "Authentification : identifiants Nagios/Icinga par défaut et faibles"
order: 1
description: "Les interfaces web de Nagios et d'Icinga s'authentifient avec un mot de passe, et les comptes par défaut et les choix faibles sont la voie d'entrée : nagiosadmin avec un mot de passe d'installation sur Core, admin ou nagiosadmin sur Nagios XI, et des mots de passe faibles ou réutilisés en général. Une connexion web donne accès aux données de monitoring et, sur XI, aux fonctionnalités et vulnérabilités qui mènent à l'exécution de code."
keywords:
  - nagiosadmin
  - admin
  - default credentials
  - weak password
  - nagios login
---

# Authentification

Les interfaces web protègent l'accès avec un mot de passe, et les comptes par défaut et identifiants faibles sont l'entrée courante. L'interface CGI de Nagios Core utilise un compte htpasswd, conventionnellement `nagiosadmin`, avec un mot de passe défini à l'installation qui est fréquemment faible ou laissé à une valeur documentée. Nagios XI est livré avec un compte admin (`admin` ou `nagiosadmin`) doté d'un mot de passe par défaut ou d'installation, et sa connexion web est la porte d'entrée vers toute l'application. Icinga Web 2 a de même un compte admin. Les mots de passe faibles et réutilisés s'appliquent aux trois. Une connexion réussie donne la vue de monitoring (inventaire des hôtes, une carte du parc) et, surtout sur Nagios XI, l'accès à la configuration et aux fonctionnalités administratives sur lesquelles reposent les vulnérabilités d'exécution de code.

```bash
# Nagios Core CGI (HTTP basic against the nagios cgi)
curl -sk -u nagiosadmin:nagiosadmin https://<target>/nagios/cgi-bin/status.cgi
# Nagios XI login (form) and API
curl -sk -c cj https://<target>/nagiosxi/login.php --data 'username=admin&password=admin&loginButton='
# spray weak/default passwords against the identified product's login
```

## Notes d'exploitation

- Essayez d'abord les comptes par défaut documentés du produit : `nagiosadmin` sur Core, `admin`/`nagiosadmin` sur XI, et les valeurs par défaut d'installation ; ceux-ci sont fréquemment inchangés.
- L'accès à Nagios XI est la connexion de plus grande valeur car l'application expose les fonctionnalités et endpoints derrière les [chaînes d'exploitation XI](nagios-xi-exploits.md) et l'[injection de commandes](command-and-plugin-injection.md) ; la connexion CGI de Core donne surtout la vue de monitoring.
- Les mots de passe faibles/réutilisés et l'absence de verrouillage sur les installations plus anciennes rendent le spraying efficace ; dérivez les candidats depuis l'organisation.
- Une connexion web oriente vers l'[injection de commandes](command-and-plugin-injection.md) et, sur XI, vers les portions authentifiées des [chaînes d'exploitation](nagios-xi-exploits.md).

## Références

- [Nagios XI administration](https://www.nagios.com/products/nagios-xi/)
- [HackTricks: Nagios](https://book.hacktricks.xyz/network-services-pentesting/5666-pentesting-nrpe)
