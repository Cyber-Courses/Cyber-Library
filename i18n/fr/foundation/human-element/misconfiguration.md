---
title: "Mauvaise configuration : quand un logiciel sûr est déployé de façon non sécurisée"
description: "L'écart entre une conception sûre et un déploiement sûr, où les paramètres par défaut et de petites négligences ouvrent discrètement la porte aux attaquants."
keywords:
  - mauvaise configuration
  - mauvaise configuration de sécurité
  - paramètres par défaut
  - durcissement
  - valeurs par défaut sécurisées
  - gestion de la configuration
---

# Mauvaise configuration

**La mauvaise configuration** est l'écart entre un système qui pourrait être sûr et un système qui l'est réellement, créé lorsqu'un logiciel est déployé avec des valeurs par défaut faibles, des fonctionnalités inutiles ou des paramètres qui n'ont jamais été renforcés. Le code peut être solide et la conception robuste, et pourtant le système en fonctionnement expose quand même des données ou des accès parce que quelqu'un a laissé un paramètre à sa valeur d'usine, a sauté une étape ou n'a pas compris ce que contrôlait une option. C'est systématiquement l'une des causes profondes les plus fréquentes des violations de données, et elle se situe clairement du côté humain de la sécurité, car elle découle de décisions et de négligences, et non d'un défaut du logiciel lui-même.

## Un code solide, des paramètres dangereux

Un produit peut réussir toutes les revues de code et pourtant être livré dans un état dangereux. C'est la ligne qui sépare la mauvaise configuration des [erreurs de conception et de développement](design-and-development-errors.md) : pour une erreur de conception, le correctif est une nouvelle version du logiciel ; pour une mauvaise configuration, le correctif est un changement dans la façon dont le logiciel existant est paramétré.

L'écart est large, car les systèmes modernes comptent des milliers de paramètres ajustables à travers les systèmes d'exploitation, les bases de données, les services cloud et les applications, et une seule valeur erronée peut réduire à néant une sécurité par ailleurs solide. Il se cache aussi très bien. Un système mal configuré fonctionne généralement sans problème pour les utilisateurs normaux, personne ne se plaint donc, et le problème reste invisible jusqu'à ce qu'une personne malveillante parte à sa recherche.

## Quatre façons dont un paramètre devient erroné

La plupart des mauvais paramètres arrivent par l'une de ces voies :

1. **Il n'a jamais été modifié.** Le fournisseur l'a livré ouvert ou avec un mot de passe connu, et l'installateur a accepté les valeurs par défaut.
2. **Il a été assoupli pour faire fonctionner quelque chose.** Une autorisation a été élargie ou une vérification désactivée lors d'un dépannage ou sous la pression d'une échéance, et n'a jamais été rétablie.
3. **Personne ne comprenait ce qu'il faisait.** Une option au nom obscur ou à l'effet de bord caché a été réglée sans connaître son impact sur la sécurité.
4. **Il a dérivé.** Le système a été durci une fois, puis des modifications manuelles, des mises à niveau et de nouveaux composants l'ont lentement éloigné de cette référence.

La première voie explique pourquoi l'initiative Secure by Design de la CISA pousse les fournisseurs à livrer des produits verrouillés par défaut, afin que l'état sûr soit celui que les clients obtiennent sans rien faire.

## Ce qu'expose la mauvaise valeur

| Paramètre laissé erroné | Ce qu'obtient un attaquant |
|---|---|
| Identifiants par défaut (admin/admin, mots de passe fournis par le fournisseur) | Une connexion directe à une console d'administration ou à un équipement |
| Compartiments de stockage, bases de données ou partages de fichiers publics | Le téléchargement massif de données, souvent repérées par des scanners automatisés |
| Applications d'exemple, modes de débogage, listage de répertoires, services inutilisés | Des chemins de code et des fichiers supplémentaires qui élargissent la [surface d'attaque](../concepts/assets-and-attack-surface.md) |
| Chiffrement disponible mais non activé | Des données lisibles en transit ou au repos |
| Messages d'erreur verbeux et traces d'exécution | Une carte des composants internes et de leurs versions |
| Paramètres hérités non sécurisés conservés pour la compatibilité ascendante | D'anciens protocoles et options plus faibles vers lesquels un attaquant peut forcer une rétrogradation |
| Comptes disposant de plus de droits que leur fonction ne l'exige | Une petite compromission qui en devient une grande (voir [Moindre privilège](../principles/least-privilege.md)) |

Aucune de ces failles ne nécessite d'outillage particulier pour être exploitée. C'est pourquoi la mauvaise configuration occupe une place si élevée dans les données du secteur : dans l'OWASP Top 10:2025, Security Misconfiguration est passée de la cinquième à la deuxième place, et l'OWASP indique que toutes les applications testées en présentaient une forme ou une autre.

## Le point de vue des équipes rouges et bleues

En 2023, la NSA et la CISA ont publié les dix mauvaises configurations que leurs équipes rouges et bleues rencontraient le plus souvent dans les grandes organisations. La liste s'ouvre sur les configurations par défaut des logiciels et des applications, suivies de la séparation inadéquate des privilèges utilisateur et administrateur, et comprend aussi une surveillance insuffisante du réseau interne, l'absence de segmentation réseau, une mauvaise gestion des correctifs, une MFA faible ou mal configurée, des contrôles d'accès insuffisants sur les partages réseau et une mauvaise hygiène des identifiants.

Deux leçons se dégagent. D'abord, il ne s'agit pas de faiblesses exotiques : ce sont les paramètres de routine que les équipes débordées remettent à plus tard. Ensuite, le cloud les fait mal passer à l'échelle, puisqu'un seul modèle ou paramètre par défaut erroné peut être copié sur des centaines de ressources en quelques secondes, transformant une simple négligence en exposition à l'échelle de toute l'organisation.

## Garder une configuration fiable dans la durée

Le durcissement n'est pas une tâche ponctuelle, car les configurations dérivent. Une approche durable traite les paramètres comme quelque chose à définir, vérifier et élaguer en continu :

- **Partir d'un état verrouillé.** Privilégiez les systèmes livrés avec des valeurs par défaut sécurisées, et traitez chaque paramètre assoupli comme un choix délibéré et documenté. C'est le volet configuration de la [sécurité dès la conception](../principles/security-by-design.md).
- **Mesurer par rapport à une référence.** Appliquez des guides de durcissement reconnus, tels que les CIS Benchmarks ou le NIST SP 800-123 pour les serveurs, plutôt que de vous fier à votre mémoire.
- **Définir les paramètres sous forme de code.** Déployez à partir de modèles révisés afin que la même référence vérifiée soit appliquée à chaque fois, et qu'une correction du modèle corrige chaque copie.
- **Faire relire les changements par une seconde personne.** La séparation des tâches permet de détecter les erreurs de bonne foi avant qu'elles n'atteignent la production.
- **Détecter les dérives.** L'analyse de configuration et de posture fait remonter les paramètres risqués et les changements non approuvés avant que les attaquants ne les trouvent.
- **Supprimer ce que vous n'utilisez pas.** Désactiver les fonctionnalités, comptes et services inutilisés réduit la surface d'attaque et le nombre de paramètres susceptibles d'être mal réglés.

Les outils ont leurs limites : les administrateurs doivent encore savoir reconnaître une option dangereuse lorsqu'ils en voient une, et c'est là qu'interviennent la [formation et la sensibilisation](training-and-awareness.md). Comme les raccourcis du quotidien décrits dans [comportement et mauvais usage des utilisateurs](user-behavior-and-misuse.md), la mauvaise configuration crée des ouvertures sans aucun bug logiciel, et un seul mauvais paramètre peut discrètement compromettre la confidentialité, l'intégrité et la disponibilité décrites dans la [Triade CIA](../concepts/cia-triad.md).

## Références

- [OWASP Top 10:2025, A02 Security Misconfiguration](https://owasp.org/Top10/2025/A02_2025-Security_Misconfiguration/)
- [NSA et CISA : Top Ten Cybersecurity Misconfigurations (AA23-278A)](https://www.cisa.gov/news-events/cybersecurity-advisories/aa23-278a)
- [MITRE CWE-16: Configuration](https://cwe.mitre.org/data/definitions/16.html)
- [NIST SP 800-123, Guide to General Server Security](https://csrc.nist.gov/pubs/sp/800/123/final)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)
- [CISA: Secure by Design](https://www.cisa.gov/securebydesign)
