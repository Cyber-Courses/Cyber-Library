---
title: "Sécurité contre vie privée : concilier la protection et les droits individuels"
description: "Comment les mesures qui protègent les systèmes et les personnes peuvent entrer en conflit avec le droit de l'individu à la vie privée, et pourquoi les deux ne sont pas de simples opposés."
keywords:
  - sécurité contre vie privée
  - vie privée
  - surveillance
  - minimisation des données
  - proportionnalité
  - éthique de la sécurité
---

# Sécurité contre vie privée

**La sécurité et la vie privée sont souvent présentées comme un compromis**, comme si tout gain dans l'une devait se faire au détriment de l'autre. La réalité est plus enchevêtrée. La sécurité est la pratique consistant à protéger les personnes, les données et les systèmes contre les préjudices ; la vie privée est la revendication de l'individu à contrôler ce que l'on sait de lui et par qui. La tension apparaît aux marges, là où une mesure destinée à assurer la sécurité des personnes les surveille aussi, les enregistre ou décide de choses à leur sujet.

## Là où les deux vont dans le même sens

Une grande partie de la sécurité existe pour protéger la vie privée. Le chiffrement rend les messages et les enregistrements stockés illisibles pour quiconque ne possède pas la clé. Le contrôle d'accès, fondé sur [l'authentification et l'autorisation](../principles/authentication-and-authorization.md), détermine qui peut voir des données personnelles. Prévenir les fuites, c'est protéger la vie privée : une base de données clients divulguée est à la fois un échec de sécurité et une atteinte à la vie privée.

C'est pourquoi les normes traitent les deux ensemble. Le NIST SP 800-53 s'intitule "Security and Privacy Controls", et le NIST Privacy Framework est conçu pour s'articuler avec la gestion des risques de sécurité plutôt que pour la concurrencer. La vraie question n'est pas de savoir quelle valeur l'emporte, mais ce qui se passe lorsqu'une mesure de protection doit observer les personnes pour fonctionner.

## Le moment où la protection se met à surveiller

Les mesures de protection recueillent de l'information. Pour détecter un intrus, on journalise l'activité ; pour arrêter la fraude, on profile les comportements ; pour bloquer les abus, on inspecte les contenus. Chacune peut être légitime, et chacune coûte un peu de vie privée. Le tableau ci-dessous présente les collisions récurrentes.

| Mesure | Ce contre quoi elle protège | Ce qu'elle expose | Alternative plus douce |
| --- | --- | --- | --- |
| Surveillance et journalisation | Intrusions, abus internes | Un enregistrement durable des comportements ordinaires, qui peut être détourné ou compromis | Journaliser ce dont les enquêtes ont besoin, pseudonymiser les identifiants, restreindre qui peut interroger les journaux |
| Conservation des données | Perte de preuves pour des enquêtes ultérieures | Tout ce qui est gardé "au cas où" accroît le préjudice lorsque ce stock est exposé | Durées de conservation fixes avec suppression automatique |
| Inspection des contenus | Logiciels malveillants, abus, fuites de données | Les communications privées de toutes les personnes analysées | Inspecter d'abord les métadonnées ou les types de fichiers, et ne passer au contenu que sur signal |
| Vérification d'identité | Imposteurs et prise de contrôle de comptes | Des attributs sensibles que les personnes préféreraient ne pas divulguer | Vérifier une affirmation (par exemple "plus de 18 ans") sans recueillir l'identité complète |

Dans chaque ligne, le conflit oppose rarement la sécurité à l'absence de sécurité. Il met généralement en balance la sûreté d'une personne et le droit d'une autre à ne pas être observée, ou un bénéfice collectif et un coût individuel.

## Cinq questions avant de déployer un contrôle qui surveille des personnes

Le droit et les lignes directrices en matière de vie privée convergent vers une courte série de tests. Les Lignes directrices de l'OCDE sur la protection de la vie privée (1980, révisées en 2013) et l'article 5 du RGPD de l'UE les expriment sous forme de principes tels que la limitation des finalités, la minimisation des données et la limitation de la conservation.

1. **Est-ce nécessaire ?** Nommez la menace précise. "Une meilleure visibilité" n'est pas une menace.
2. **Est-ce proportionné ?** Comparez l'intrusion au préjudice évité, et demandez-vous si une mesure moins intrusive ferait presque aussi bien.
3. **Quelles sont les données minimales ?** Ne collectez que ce qui sert la finalité déclarée (voir la [minimisation](../principles/minimization.md)).
4. **Qui peut les voir, et pendant combien de temps ?** Limitez l'accès à ceux qui en ont besoin et supprimez selon un calendrier.
5. **La finalité est-elle figée ?** Décidez à l'avance à quoi les données ne serviront pas, et mettez-le par écrit.

Un contrôle qui réussit les cinq tests n'est pas automatiquement éthique, mais un contrôle qui échoue à l'un d'eux fait peser sur la vie privée un coût qu'il n'a pas justifié.

## Pourquoi l'équilibre paraît rarement équitable

Cette liste suppose un décideur neutre. En pratique, plusieurs forces font pencher le résultat vers davantage de collecte.

- **La proportionnalité est un jugement, pas une formule.** Déterminer quel niveau de surveillance est "suffisant" pour une menace donnée est discutable, et les personnes qui en supportent le coût pour leur vie privée ne sont souvent pas celles qui décident.
- **Le consentement est fréquemment absent ou contraint.** Les employés, les citoyens et les utilisateurs ont rarement une véritable possibilité de refuser la surveillance intégrée aux systèmes dont ils dépendent, ce qui explique pourquoi [l'autorisation et le consentement](authorization-and-consent.md) se trouvent au cœur de tant de différends.
- **La dérive des usages.** Les données recueillies à une fin de protection ont tendance à être réutilisées plus tard pour quelque chose que la personne concernée n'a jamais accepté.
- **L'asymétrie de pouvoir.** Ceux qui déploient des mesures de sécurité détiennent généralement bien plus d'information et de levier que ceux qui y sont soumis, si bien que le compromis semble imposé plutôt que négocié.

La même tension apparaît sous l'angle de l'autonomie dans [liberté contre contrôle](freedom-versus-control.md), et la question de savoir qui répond des coûts imposés à la vie privée d'autrui est abordée dans [responsabilité morale](moral-responsibility.md). Les outils de surveillance sont aussi un cas d'école de [connaissance à double usage](dual-use-knowledge.md) : la capacité qui protège un réseau peut être retournée contre les personnes qui l'utilisent. La technologie ne cesse de changer, mais cet affrontement entre protection et autonomie ne se résout pas de lui-même.

## Références

- [NIST Privacy Framework](https://www.nist.gov/privacy-framework)
- [NIST SP 800-53 Rev. 5, Security and Privacy Controls for Information Systems and Organizations](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [OECD Recommendation concerning Guidelines Governing the Protection of Privacy and Transborder Flows of Personal Data](https://legalinstruments.oecd.org/en/instruments/OECD-LEGAL-0188)
- [RGPD, article 5, Principes relatifs au traitement des données à caractère personnel](https://gdpr-info.eu/art-5-gdpr/)
- [Electronic Frontier Foundation : Privacy](https://www.eff.org/issues/privacy)
