---
title: "Comportement des utilisateurs et mésusage : les habitudes quotidiennes qui créent du risque"
description: "Comment les choix ordinaires, les raccourcis et les habitudes des utilisateurs légitimes exposent les systèmes au risque, et pourquoi corriger l'environnement vaut généralement mieux que blâmer la personne."
keywords:
  - comportement des utilisateurs
  - sensibilisation à la sécurité
  - risque interne
  - erreur humaine
  - shadow it
  - usage acceptable
---

# Comportement des utilisateurs et mésusage

**La plupart des incidents de sécurité remontent à un utilisateur légitime qui fait un choix ordinaire**, et non à un génie isolé cassant la cryptographie. Les personnes qui utilisent un système au quotidien détiennent déjà l'accès, le contexte et la confiance autour desquels les défenses sont construites ; leurs habitudes, bonnes et mauvaises, façonnent donc la véritable surface d'attaque plus que n'importe quel contrôle isolé.

## Un continuum de l'erreur honnête au préjudice délibéré

Le "mésusage" recouvre un large éventail, et la cause compte, car chaque point du continuum appelle une réponse différente. Les regrouper, et traiter chaque faux pas comme s'il s'agissait d'un sabotage, apprend aux gens à cacher leurs erreurs au lieu de les signaler.

| Point du continuum | Exemple quotidien | Ce qui le motive | Ce qui aide réellement |
|---|---|---|---|
| Erreur honnête | Envoyer un fichier par e-mail au mauvais "Sam" | Fatigue, interfaces déroutantes | Valeurs par défaut plus sûres, confirmation et annulation, signalement sans blâme |
| Raccourci de confort | Enregistrer son travail dans un cloud personnel | Friction, délais, outils manquants | Corriger le processus ; fournir un outil approuvé tout aussi simple |
| Action manipulée | Payer une facture "urgente" convaincante | Un attaquant exploitant la confiance | Habitudes de vérification, contrôles hors bande |
| Mésusage délibéré | Copier des données clients avant de démissionner | Gain ou grief | Moindre privilège, surveillance, départ encadré |

Seule la dernière ligne constitue une menace interne au sens hostile. Les trois premières sont le cas courant, et elles répondent bien mieux à une meilleure conception qu'à des avertissements sévères.

## Vivre à l'intérieur de la frontière de confiance

Ce qui rend le comportement des utilisateurs si puissant, c'est la position. Le compte d'un utilisateur se situe à l'intérieur de la frontière de confiance, déjà authentifié et déjà autorisé ; un faux pas depuis ce compte contourne donc les défenses périmétriques qui arrêteraient net un intrus. Quelques habitudes reviennent dans presque toutes les organisations :

- **Réutiliser les mots de passe**, de sorte qu'une fuite sans rapport déverrouille un compte professionnel.
- **Agir sur une manipulation d'ingénierie sociale**, en cliquant, en ouvrant ou en approuvant avant de vérifier.
- **Le shadow IT**, en adoptant des applications non examinées pour aller plus vite.
- **Le partage excessif**, en utilisant "toute personne disposant du lien" là où un partage nominatif suffirait.
- **Désactiver les protections** jugées gênantes, des mises à jour aux verrouillages d'écran.

Aucune de ces habitudes n'est exotique, et c'est bien là le point : les attaquants préfèrent le chemin fiable.

## Déroulé : un seul mot de passe réutilisé

Un seul mot de passe réutilisé montre comment de petits choix s'enchaînent jusqu'à une fuite.

1. Une analyste réutilise son mot de passe professionnel sur un forum de loisirs.
2. Le forum est compromis et ses identifiants sont diffusés en ligne.
3. Un attaquant rejoue ce couple e-mail/mot de passe contre la messagerie web de l'entreprise, qui n'a pas de second facteur, et parvient à entrer.
4. Dans la boîte aux lettres, il trouve un lien de dossier rendu public des mois plus tôt pour un prestataire.
5. Des données quittent l'entreprise, inaperçues pendant des semaines car rien ne surveille les téléchargements en masse.

Aucun des choix de l'analyste n'était malveillant, et rien dans la chaîne n'avait l'air dramatique ; le seul préjudice délibéré est venu de l'attaquant extérieur qui a rejoué le mot de passe volé et emporté les données. Briser la chaîne n'importe où, un gestionnaire de mots de passe, un second facteur, un lien qui expire, une alerte sur un accès inhabituel, aurait suffi à l'arrêter.

## Traiter les contournements comme un retour, pas comme de la désobéissance

Quand beaucoup de personnes contournent un contrôle, c'est généralement le contrôle qui pose problème. Le contournement est un signal qui pointe vers une friction à corriger. Les réponses les plus solides changent l'environnement : faire du chemin sûr le chemin facile, et limiter la portée de toute erreur avec le [moindre privilège](../principles/least-privilege.md) et la segmentation. C'est pourquoi le comportement est indissociable des réglages dont les gens héritent, abordés dans la [mauvaise configuration](misconfiguration.md), et des valeurs par défaut livrées dans les logiciels, abordées dans les [erreurs de conception et de développement](design-and-development-errors.md). Les habitudes durables viennent d'une [formation et sensibilisation](training-and-awareness.md) courte et adaptée au rôle, assortie d'un moyen rapide et sans blâme de signaler, et non d'un diaporama annuel.

## Références

- [NIST SP 800-50 Rev. 1, Building a Cybersecurity and Privacy Learning Program](https://csrc.nist.gov/pubs/sp/800/50/r1/final)
- [CISA: Avoiding Social Engineering and Phishing Attacks](https://www.cisa.gov/news-events/news/avoiding-social-engineering-and-phishing-attacks)
- [Verizon Data Breach Investigations Report](https://www.verizon.com/business/resources/reports/dbir/)
- [MITRE ATT&CK: Phishing (T1566)](https://attack.mitre.org/techniques/T1566/)
- [NIST SP 800-63B-4, Digital Identity Guidelines: Authenticator Management](https://pages.nist.gov/800-63-4/sp800-63b.html)
