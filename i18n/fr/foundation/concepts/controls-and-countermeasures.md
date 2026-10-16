---
title: "Contrôles et contre-mesures : préventifs, détectifs et correctifs"
description: "Comment les contrôles préventifs, détectifs et correctifs fonctionnent ensemble pour réduire le risque, et les différentes formes qu'une même protection peut prendre."
keywords:
  - contrôles de sécurité
  - contre-mesures
  - contrôles préventifs
  - contrôles détectifs
  - contrôles correctifs
  - contrôles compensatoires
---

# Contrôles et contre-mesures

**Un contrôle** est toute protection ou mesure mise en place pour réduire le risque pesant sur un actif, et une **contre-mesure** est la même idée nommée du point de vue de l'attaquant : une réponse visant une menace précise. Les deux termes sont employés de façon presque interchangeable. Ce qui compte, c'est le rôle qu'ils jouent. Un contrôle ne fait pas disparaître une faiblesse ; il réduit la probabilité qu'elle soit exploitée, les dommages si elle l'est, ou le délai avant que quelqu'un s'en aperçoive. La sécurité, c'est l'art de choisir un ensemble sensé de ces mesures et de les agencer pour qu'elles se renforcent mutuellement.

## Prévenir, détecter, corriger

La manière la plus utile de classer les contrôles est selon le moment où ils agissent par rapport à un incident. Chaque fonction répond à une question différente, et une défense sérieuse a besoin des trois.

| Fonction | Question à laquelle elle répond | Exemples |
| --- | --- | --- |
| Préventive | Comment empêcher que cela arrive ? | Contrôles d'accès, chiffrement, pare-feu, validation des entrées, formation du personnel |
| Détective | Comment savoir que cela se produit ou s'est produit ? | Journalisation, détection d'intrusion, surveillance de l'intégrité des fichiers, audits |
| Corrective | Comment se rétablir une fois que c'est arrivé ? | Sauvegardes et restaurations, application de correctifs, réponse aux incidents, isolement d'un hôte |

Une quatrième fonction, **dissuasive**, décourage un acteur avant qu'il ne tente quelque chose (caméras visibles, bannières d'avertissement, perspective de poursuites judiciaires). La prévention n'est jamais parfaite, et c'est précisément pour cela que les contrôles détectifs et correctifs existent : ils partent du principe que certaines attaques passeront et raccourcissent le temps pendant lequel un attaquant reste inaperçu et incontrôlé.

## Administratifs, techniques, physiques

La fonction ci-dessus indique ce que fait un contrôle ; un second axe indique comment il est bâti. Un même objectif peut être atteint sous chacune de ces formes, et les programmes matures les combinent plutôt que de s'appuyer sur une seule.

- Les contrôles **administratifs** sont des politiques, des procédures et des formations : les règles que les personnes sont censées suivre, comme une politique de revue des accès ou une règle de bureau propre.
- Les contrôles **techniques** (ou logiques) sont imposés par le matériel ou le logiciel : une exigence de mot de passe, le chiffrement de disque, une règle de segmentation réseau.
- Les contrôles **physiques** agissent sur le monde tangible : portes verrouillées, lecteurs de badges, câbles antivol, destruction sécurisée des disques.

Verrouiller une salle serveurs (physique), exiger une authentification (technique) et interdire les comptes partagés par une politique (administratif) peuvent tous servir le même objectif : tenir les mauvaises personnes à l'écart d'une machine.

## Quand le contrôle principal ne peut pas être utilisé

Parfois, le contrôle recommandé est impossible, trop coûteux, ou casse quelque chose. Un **contrôle compensatoire** est un substitut délibéré qui offre une protection comparable par une autre voie. Si un ancien système ne peut pas prendre en charge l'authentification multifacteur, le placer sur un segment de réseau isolé et surveiller ses journaux de près peut couvrir une grande partie du même risque. Les contrôles compensatoires sont légitimes, mais ils constituent un compromis, pas un laissez-passer : chacun doit être documenté, et sa couverture plus faible comprise, plutôt que silencieusement acceptée.

## Un ordinateur portable volé, contrôle par contrôle

Imaginez l'ordinateur portable d'un employé dérobé dans une voiture. Suivez les contrôles dans l'ordre :

1. **Préventif.** Le chiffrement intégral du disque fait que, tant que l'ordinateur reste éteint, le voleur ne voit que du texte chiffré, de sorte que les données restent confidentielles même si le matériel a disparu.
2. **Détectif.** L'employé signale la perte et les journaux de gestion des appareils confirment que l'ordinateur ne s'est pas reconnecté, le signalant comme manquant.
3. **Correctif.** Les administrateurs révoquent ses identifiants et déclenchent un effacement à distance, et l'utilisateur restaure ses fichiers depuis une sauvegarde sur un appareil de remplacement.

Aucune mesure isolée n'a géré l'incident. Le chiffrement a gagné du temps, le signalement et les journaux l'ont détecté, et l'effacement et la restauration l'ont clôturé.

## Comment les contrôles se relient au risque et à la défense

Les contrôles existent pour protéger la confidentialité, l'intégrité et la disponibilité nommées dans la [triade CIA](cia-triad.md), en réduisant les occasions offertes aux [menaces et acteurs de menace](threats-and-threat-actors.md) qui ciblent une organisation. Chaque contrôle traite une ou plusieurs [vulnérabilités et exploits](vulnerabilities-and-exploits.md), et superposer des contrôles indépendants de sorte que la défaillance de l'un n'expose pas tout est le principe de la [défense en profondeur](../principles/defense-in-depth.md).

## Références

- [NIST SP 800-53 Rev. 5, Security and Privacy Controls for Information Systems and Organizations](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [NIST Glossary: Security Control](https://csrc.nist.gov/glossary/term/security_control)
- [NIST Glossary: Countermeasures](https://csrc.nist.gov/glossary/term/countermeasures)
- [CIS Critical Security Controls](https://www.cisecurity.org/controls)
- [ISO/IEC 27002, Information security controls](https://www.iso.org/standard/75652.html)
