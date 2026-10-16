---
title: "Comportement des utilisateurs et mauvais usage : comment les habitudes quotidiennes créent du risque"
description: "Comment les choix, les raccourcis et les habitudes quotidiennes des utilisateurs légitimes exposent les systèmes au risque, et les mesures concrètes qui réduisent cette exposition."
keywords:
  - comportement des utilisateurs
  - sensibilisation à la sécurité
  - risque interne
  - erreur humaine
  - shadow it
  - usage acceptable
---

# Comportement des utilisateurs et mauvais usage

**Le comportement des utilisateurs** est l'un des facteurs les plus importants et les moins prévisibles en matière de sécurité, car les personnes qui utilisent un système au quotidien détiennent aussi les accès, le contexte et la confiance autour desquels les défenses sont construites. La plupart des incidents ne commencent pas par un exploit exotique. Ils commencent par une personne ordinaire qui fait un choix d'apparence raisonnable : réutiliser un mot de passe, cliquer sur un lien, partager un fichier ou contourner un contrôle qui la gênait.

## Erreur, raccourci ou sabotage

Le mauvais usage signifie rarement de la malveillance. Bien plus souvent, il s'agit d'un raccourci pris sous la pression du temps, d'une habitude héritée des appareils personnels, ou d'un écart entre ce que disent les règles et ce que le travail exige réellement. Il est important de distinguer ces cas, car chacun appelle une réponse différente.

| Type de comportement | Exemple typique | Ce qui le motive | Réponse efficace |
|---|---|---|---|
| Erreur de bonne foi | Envoyer un tableur au mauvais destinataire | Inattention, fatigue, interfaces déroutantes | Réglages par défaut plus sûrs, annulation et étapes de confirmation, signalement sans reproche |
| Raccourci délibéré | Stocker des fichiers professionnels sur un compte cloud personnel | Friction, échéances, outils manquants | Corriger le processus ou fournir un outil approuvé tout aussi simple |
| Action manipulée | Valider un paiement après un e-mail "urgent" convaincant | Un attaquant qui exploite la confiance et l'urgence | Réflexes de vérification, contrôles hors bande, authentification résistante au phishing |
| Mauvais usage malveillant | Copier des données clients avant de quitter l'entreprise | Gain personnel ou rancœur | Moindre privilège, surveillance, gestion des départs, responsabilisation |

Seule la dernière ligne relève de la menace interne au sens hostile. Traiter les trois premières comme la quatrième engendre la peur, et la peur pousse les gens à cacher leurs erreurs au lieu de les signaler.

## Cinq habitudes qui donnent une longueur d'avance aux attaquants

Les utilisateurs opèrent à l'intérieur de la frontière de confiance : leurs comptes disposent déjà d'un accès légitime, de sorte que leurs erreurs contournent de nombreuses défenses périmétriques qui arrêteraient un attaquant extérieur. Ces habitudes reviennent sans cesse :

- **Mauvaises habitudes en matière d'identifiants :** réutiliser des mots de passe sur plusieurs sites, en choisir de faciles à deviner, ou les noter là où d'autres peuvent les trouver. Une personne qui réutilise un mot de passe est un risque ; toute une organisation qui le fait est une faiblesse systémique qu'un seul identifiant divulgué peut déverrouiller.
- **Céder à l'ingénierie sociale :** cliquer sur des liens malveillants, ouvrir des pièces jointes inattendues, ou donner suite à des demandes urgentes sans les vérifier. Le phishing et le pretexting existent précisément parce qu'influencer une personne est souvent plus facile que de vaincre un contrôle technique.
- **Shadow IT :** adopter des applications non approuvées, du stockage cloud ou des comptes personnels pour travailler plus vite, en dehors de tout examen de sécurité.
- **Partage excessif de données :** envoyer des fichiers sensibles vers des adresses personnelles, utiliser des partages publics "toute personne disposant du lien", ou accorder des accès bien plus largement que nécessaire.
- **Désactivation des protections :** désactiver les mises à jour, couper le verrouillage d'écran, ou trouver des moyens de contourner les invites d'authentification multifacteur jugées pénibles.

## Comment un raccourci devient un incident

Un scénario concret montre comment des choix ordinaires s'enchaînent :

1. Une analyste réutilise son mot de passe professionnel sur un forum consacré à un loisir, parce que retenir un mot de passe de plus lui semble une corvée.
2. Le forum est piraté et sa liste de mots de passe est publiée.
3. Un attaquant teste le couple adresse e-mail et mot de passe divulgué sur la messagerie web de l'entreprise. Cela fonctionne, car le compte n'a pas d'authentification multifacteur.
4. Depuis la boîte aux lettres, l'attaquant trouve le lien d'un dossier partagé rendu public des mois plus tôt pour aider un prestataire.
5. Des données clients quittent l'entreprise, et personne ne s'en aperçoit pendant des semaines, car personne ne surveille les téléchargements inhabituels.

Aucune étape n'a exigé de malveillance de la part de l'analyste, et aucune ne paraît spectaculaire prise isolément. Rompre la chaîne à n'importe quel point (un gestionnaire de mots de passe, une authentification multifacteur, un lien qui expire, une alerte sur un accès massif) l'aurait arrêtée. C'est pourquoi le comportement des utilisateurs est si étroitement lié aux [erreurs de conception et de développement](design-and-development-errors.md) et à la [mauvaise configuration](misconfiguration.md) : les réglages dont héritent les personnes déterminent jusqu'où une erreur peut se propager.

## Les contournements sont un signal de conception

Lorsque de nombreux utilisateurs contournent un contrôle, c'est généralement que le contrôle est trop contraignant, et le contournement signale un processus à corriger plutôt qu'une personne à blâmer. Les mesures les plus efficaces modifient l'environnement autant que l'individu :

- **Faites du chemin sécurisé le chemin le plus simple.** Les gestionnaires de mots de passe, l'authentification unique (single sign-on) et des réglages par défaut judicieux suppriment la friction qui pousse aux contournements.
- **Réduisez ce qu'une erreur peut atteindre.** Le [moindre privilège](../principles/least-privilege.md) et la segmentation limitent les dégâts que peut causer un compte lorsqu'il est détourné ou compromis.
- **Construisez des habitudes durables, pas des leçons ponctuelles.** Des actions de [formation et de sensibilisation](training-and-awareness.md) courtes, fréquentes et adaptées aux rôles, avec une pratique réaliste, valent mieux qu'un diaporama annuel.
- **Offrez un moyen rapide et sans reproche de signaler.** Lorsque les personnes peuvent signaler une erreur ou un message suspect sans crainte, les problèmes apparaissent tant qu'ils sont encore mineurs.
- **Surveillez les schémas qui comptent.** Alertez sur les accès inhabituels et les actions à risque afin que le mauvais usage soit détecté rapidement, tout en respectant la vie privée et la proportionnalité.

## Références

- [NIST SP 800-50 Rev. 1, Building a Cybersecurity and Privacy Learning Program](https://csrc.nist.gov/pubs/sp/800/50/r1/final)
- [CISA: Avoiding Social Engineering and Phishing Attacks](https://www.cisa.gov/news-events/news/avoiding-social-engineering-and-phishing-attacks)
- [Verizon Data Breach Investigations Report](https://www.verizon.com/business/resources/reports/dbir/)
- [MITRE ATT&CK: Phishing (T1566)](https://attack.mitre.org/techniques/T1566/)
- [NIST SP 800-63B-4, Digital Identity Guidelines: Authentication and Authenticator Management](https://pages.nist.gov/800-63-4/sp800-63b.html)
