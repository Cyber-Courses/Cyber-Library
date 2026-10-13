---
title: "XXE aveugle sans sortie reflétée"
description: "Quand le parseur résout les entités mais ne renvoie rien de parsé, les données sont récupérées via les messages d'erreur, le timing et les canaux hors bande."
keywords:
  - XXE aveugle
  - aucune sortie reflétée
  - XXE basé sur les erreurs
  - canal hors bande
  - entité paramètre
---

# Aveugle

Le XXE aveugle couvre le cas courant où l'application parse le XML de l'attaquant avec les entités externes activées mais ne place jamais aucune valeur parsée dans la réponse. La lecture en bande de `&xxe;` ne produit rien, parce que le champ où elle atterrirait n'est pas renvoyé, est écarté après validation, ou est consommé par un backend qui ne renvoie qu'un statut.

La vulnérabilité reste pleinement exploitable ; les données doivent simplement sortir par un canal auxiliaire. Trois canaux font le travail. La récupération **hors bande** fait envoyer au parseur le contenu du fichier vers un serveur contrôlé par l'attaquant via HTTP ou FTP. Les **messages d'erreur** poussent le parseur à inclure le contenu du fichier dans une chaîne d'erreur fatale que l'application fait remonter par hasard. Le **timing** distingue les résultats quand même les erreurs sont supprimées. Tous reposent sur les **entités paramètres** et, dans la forme portable, sur une DTD externe, parce que l'imbrication d'entités générales est rejetée dans le sous-ensemble interne par la plupart des parseurs.

Ce sous-arbre couvre le canal basé sur les erreurs et la technique en DTD locale pour les environnements où l'accès réseau sortant est bloqué.

## Pages

- **[Messages d'erreur](error-messages.md)** : une entité paramètre qui alimente le contenu d'un fichier dans un chemin SYSTEM invalide force une erreur de parsing dont le message fuite le fichier.
- **[DTD locale](local-dtd.md)** : réutiliser un fichier DTD déjà présent sur le disque permet à un XXE aveugle de redéfinir une entité interne et de déclencher une fuite basée sur les erreurs quand le réseau sortant est bloqué.

## Outils

- **XXEinjector** : automatisation du XXE aveugle via des canaux hors bande et basés sur les erreurs.
- **Burp Collaborator** : capture des rappels hors bande qui confirment le XXE aveugle.
- **Burp Suite** : forge de charges utiles d'entités paramètres dans Repeater.

## Références

- PortSwigger Web Security Academy : Blind XXE injection
- OWASP : XML External Entity Prevention Cheat Sheet
