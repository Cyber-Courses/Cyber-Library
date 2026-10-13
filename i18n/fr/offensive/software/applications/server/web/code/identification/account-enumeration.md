---
title: "Énumération de comptes : oracles d'existence dans les erreurs, le statut, la forme de réponse et le timing"
description: "Comment confirmer quels comptes existent en comparant les réponses de l'application à travers la connexion, l'inscription, la réinitialisation de mot de passe et le SSO, incluant les oracles de message, de statut, de corps, de redirection et de timing."
keywords:
  - énumération de comptes
  - énumération d'utilisateurs
  - énumération de noms d'utilisateur
  - attaque par timing
  - énumération par réinitialisation de mot de passe
---

# Énumération de comptes

L'énumération de comptes consiste à confirmer si un identifiant donné (nom d'utilisateur, e-mail, téléphone) correspond à un compte réel, avant et sans s'authentifier. L'application devient un **oracle d'existence** dès que son comportement diffère entre un identifiant valide et un identifiant invalide. La sortie est une liste de comptes confirmés qui alimente le credential stuffing, le password spraying et le phishing ciblé.

## Oracles à tester

La différence entre « le compte existe » et « le compte n'existe pas » fuit à travers plusieurs canaux. Testez chacun en envoyant une requête avec un identifiant connu valide et une avec un identifiant connu invalide, en gardant tout le reste identique, et en comparant les résultats.

### Différences de message d'erreur

L'oracle classique est un texte distinct :

- **Connexion** : « Nom d'utilisateur inconnu » contre « Mot de passe incorrect » vous indique que le nom d'utilisateur est valide sur le second message. Même un « identifiants invalides » générique peut fuir ailleurs.
- **Inscription** : « Cet e-mail est déjà enregistré » confirme directement l'existence. L'inscription est souvent l'oracle le plus fiable parce qu'elle doit informer l'utilisateur des collisions.
- **Réinitialisation de mot de passe** : une application qui dit « Aucun compte avec cet e-mail » pour une entrée invalide mais « Consultez votre boîte de réception » pour une entrée valide est un oracle, même quand elle tente d'être discrète.

### Différences de statut, de corps et d'en-tête

Lorsque les messages sont uniformes, des indices plus subtils demeurent :

- **Code de statut HTTP** : 200 contre 302, ou 200 contre 400.
- **Longueur de réponse** : quelques octets de différence (un champ caché, un jeton différent, un changement d'espace) suffisent ; triez les résultats de Burp Intruder par longueur.
- **Erreurs de validation au niveau des champs** dans les API JSON : `{"error":"user_not_found"}` contre `{"error":"invalid_password"}`.
- **Set-Cookie ou cible de redirection** : une connexion valide peut définir un cookie MFA en attente ou rediriger vers `/mfa`, tandis qu'une invalide retourne vers `/login`.

### Oracles de branchement de flux

Les flux multi-étapes se ramifient selon l'existence :

- **Invite MFA** : si la page du second facteur n'apparaît que pour les comptes valides (parce que les invalides échouent à l'étape un), l'invite elle-même est l'oracle.
- **SSO / magic link** : « nous vous avons envoyé un lien par e-mail » pour les utilisateurs valides contre une erreur immédiate pour les inconnus.
- Les flux de **mot de passe oublié** et d'invitation confirment souvent l'existence explicitement.

### Oracles de timing

Lorsque les réponses sont identiques octet pour octet, le timing ne l'est souvent pas. Si le serveur exécute un hachage de mot de passe coûteux (bcrypt, argon2) uniquement lorsque le nom d'utilisateur existe, les noms d'utilisateur valides sont mesurablement plus lents ; si le serveur court-circuite sur un utilisateur inconnu, les invalides sont plus rapides. Collectez de nombreux échantillons par identifiant pour moyenner le jitter réseau, puis comparez les distributions plutôt que des requêtes uniques.

## Exploitation

Construisez une liste de candidats, puis pilotez l'oracle choisi avec un outil d'automatisation. Un oracle d'erreur de connexion avec Burp Intruder ou ffuf :

```bash
ffuf -w users.txt -u https://target/login -X POST \
  -d 'username=FUZZ&password=Wrong123!' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -mr 'Incorrect password'      # match the "valid user, wrong password" branch
```

Faire correspondre le message de l'utilisateur valide (`-mr`) renvoie exactement les comptes confirmés. Lorsque l'indice est la longueur plutôt que le texte, filtrez sur la taille (`-fs`/`-ms`). Pour les oracles de réinitialisation ou d'inscription, pointez la même technique vers `/forgot-password` ou `/register` et faites correspondre la branche qui confirme l'existence.

Pour un oracle de timing, prenez de nombreux échantillons par identifiant et agrégez avant de classer ; une seule requête n'a aucun sens face au jitter réseau. Garder le **minimum** de nombreux échantillons est un agrégat simple et robuste, puisque le jitter ne fait toujours qu'ajouter du délai :

```bash
for u in $(cat users.txt); do
  min=$(for i in $(seq 1 20); do
          curl -s -o /dev/null -w '%{time_total}\n' -X POST https://target/login \
            -d "username=$u&password=Wrong123!"
        done | sort -n | head -1)
  echo "$min  $u"
done | sort -rn | head
```

Chaque identifiant est mesuré 20 fois et réduit à sa réponse la plus rapide (la moins affectée par le jitter) ; les utilisateurs dont le plancher est systématiquement plus élevé sont ceux qui exécutent le hachage coûteux, et se classent donc en tête. Augmentez le nombre d'échantillons sur les liens bruyants et comparez les distributions, pas les valeurs ponctuelles.

L'énumération est souvent possible sur plusieurs endpoints à la fois ; l'oracle le plus fiable sur une cible donnée est fréquemment l'inscription ou la réinitialisation plutôt que la connexion, donc testez-les tous avant de conclure que l'application est sûre.

## Outils

- **Burp Suite Intruder** : piloter l'oracle, trier par statut/longueur/timing.
- **ffuf** / **wfuzz** : correspondance et filtrage rapides sur le message ou la taille.
- **Collecte de noms d'utilisateur** : profils publics, `theHarvester`, endpoints de recherche et d'autocomplétion, corpus de fuites.

## Références

- PortSwigger Web Security Academy : [Username enumeration](https://portswigger.net/web-security/authentication/password-based)
- OWASP WSTG : [Testing for Account Enumeration and Guessable User Account (WSTG-IDNT-04)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account)
