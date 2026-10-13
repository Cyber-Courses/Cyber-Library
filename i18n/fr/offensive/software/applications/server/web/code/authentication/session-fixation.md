---
title: "Attaquer la fixation de session : planter un ID de session connu et l'absence de rotation à la connexion"
description: "Comment un pentester fixe l'identifiant de session d'une victime avant la connexion et chevauche la session authentifiée : vecteurs de plantage, la distinction permissif contre strict, et un pas-à-pas détaillé."
keywords:
  - fixation de session
  - gestion des sessions
  - cookie tossing
  - détournement de session
  - JSESSIONID
---

# Fixation de session

Une technique de détournement de session qui n'a besoin d'aucun ID de session volé, car l'attaquant **l'a choisi à l'avance**. L'attaquant plante un identifiant de session connu dans le navigateur de la victime, la victime s'authentifie, et parce que l'application ne parvient pas à émettre un nouvel identifiant à la connexion, la session désormais authentifiée porte toujours la valeur connue de l'attaquant. L'attaquant la chevauche jusque dans le compte de la victime.

## Vue d'ensemble

La fixation de session est l'inverse du détournement de session : dans le détournement, l'attaquant exfiltre un ID généré par le serveur *après* la connexion ; dans la fixation, l'attaquant plante un ID *avant* la connexion et attend. L'unique cause racine est que **l'identifiant de session n'est pas régénéré lors du changement de privilège à l'authentification**, de sorte que la même valeur franchit la frontière non authentifié vers authentifié.

Deux comportements serveur déterminent l'effort de l'attaquant (la distinction originale de Kolšek) :

- Les serveurs **permissifs** acceptent des ID arbitraires fournis par le client et créent une session pour eux (PHP fonctionne ainsi par défaut). L'attaquant invente simplement un ID tel que `1234`.
- Les serveurs **stricts** n'honorent que les ID qu'ils ont générés. L'attaquant doit d'abord demander l'application sans authentification pour obtenir un véritable ID de session « piège », puis le maintenir en vie (des requêtes périodiques défont le timeout d'inactivité) jusqu'à ce que la victime se connecte.

## Exploitation

### Étape 1 - obtenir ou choisir l'ID piège

Cible permissive : choisissez n'importe quelle valeur. Cible stricte : interrogez l'application sans authentification, capturez le `Set-Cookie: SESSIONID=…` émis, et gardez-le chaud avec des requêtes périodiques jusqu'à son usage. Le timeout absolu et les redémarrages du serveur bornent la fenêtre d'attaque.

### Étape 2 - le planter dans le navigateur de la victime

Les règles de même origine impliquent que le propre domaine de l'attaquant ne peut pas définir un cookie pour la cible ; le plantage utilise donc un vecteur d'origine cible :

- **Paramètre d'URL** (lorsque l'application accepte des ID de session depuis la chaîne de requête, par exemple la réécriture d'URL de Java) : `https://app.site/login;jsessionid=1234` ou `?sessionid=1234`. Le plus simple, mais visible et facilement journalisé.
- **Script côté client via XSS** (pas de `HttpOnly`) : reflétez `document.cookie="sessionid=1234"` à travers un sink d'origine cible ; ajoutez `Expires=` pour le transformer en cookie durable et étendre la fenêtre.
- **`<meta http-equiv="Set-Cookie">`** injecté dans une page d'origine cible.
- **Cookie tossing de sous-domaine** : depuis un hôte frère plus faible, définissez un cookie porté sur le domaine parent pour que le navigateur l'envoie à la cible, `document.cookie="sessionid=1234;domain=.site"`. Un seul XSS ou une seule injection d'en-tête sur *n'importe quel* sous-domaine compromet tout le domaine.
- **Injection d'en-tête de réponse** (HTTP response splitting, ou un proxy/CDN défaillant) pour émettre `Set-Cookie` directement.

### Étape 3 - chevaucher la session authentifiée

Attendez que la victime se connecte. Parce que l'ID n'a pas été renouvelé, la session authentifiée porte toujours `1234` ; accédez-y avec `Cookie: sessionid=1234`.

### Le confirmer en tant que pentester (boîte noire)

Enregistrez le cookie émis **avant connexion**, soumettez des identifiants valides, et inspectez la réponse : si le serveur renvoie le **même** identifiant de session sans nouveau `Set-Cookie`, la fixation est confirmée. La procédure de cookie forcé du WSTG utilise deux comptes : sauvegardez les cookies avant connexion de la victime (sautez ceux préfixés par `__Host-`/`__Secure-`), authentifiez-vous, restaurez les cookies sauvegardés, et vérifiez si une action protégée fonctionne toujours sous l'identité de la victime.

## Faiblesses de gestion de session connexes

- **Pas d'invalidation côté serveur à la déconnexion** : effacer uniquement le cookie client laisse l'ID fixé actif ; un vrai correctif supprime la session côté serveur (et idéalement « déconnecte partout »).
- **Timeout absolu trop long ou absent** : laisse l'attaquant garder une session piège en vie et chevaucher une session entamée.
- **ID prévisibles / à faible entropie** : laissent un attaquant deviner des ID piège valides même sur les serveurs stricts.
- **Absence de `Secure`/`HttpOnly`/`SameSite`** : ceux-ci ne causent pas la fixation mais permettent l'étape de plantage (accès par script, injection réseau, livraison cross-site).

## Outils

- **Burp Suite** : observer `Set-Cookie` à travers la frontière de connexion ; les règles de gestion de session et les macros pour scripter la comparaison avant/après connexion.

## Références

- OWASP : [Session fixation](https://owasp.org/www-community/attacks/Session_fixation) et la [Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- OWASP WSTG : [Testing for Session Fixation (WSTG-SESS-03)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/06-Session_Management_Testing/03-Testing_for_Session_Fixation)
- Mitja Kolšek / ACROS Security : [Session Fixation Vulnerability in Web-based Applications](https://acrossecurity.com/papers/session_fixation.pdf) (l'article fondateur)
