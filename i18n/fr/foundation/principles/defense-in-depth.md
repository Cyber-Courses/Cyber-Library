---
title: "Défense en profondeur : des contrôles en couches qui échouent en toute sécurité"
description: "Le principe de sécurité consistant à empiler plusieurs contrôles indépendants afin qu'aucune défaillance isolée ne puisse exposer l'ensemble du système."
keywords:
  - défense en profondeur
  - sécurité en couches
  - contrôles de sécurité
  - redondance
  - rayon d'impact
  - principes de sécurité
---

# Défense en profondeur

**La défense en profondeur** est la pratique consistant à protéger un système avec plusieurs couches de contrôle indépendantes, de sorte que si une couche échoue ou est contournée, d'autres restent en place pour arrêter ou ralentir un attaquant. Elle part d'une hypothèse inconfortable : tout contrôle finit par échouer, à cause d'un bogue, d'une mauvaise configuration, d'un contournement astucieux ou d'une personne qui passe une mauvaise journée. L'objectif n'est pas un mur parfait, mais une conception dans laquelle aucune défaillance isolée ne décide de l'issue.

## Des murailles de forteresse aux barrières en couches

L'idée est empruntée à la stratégie militaire, où un seul mur est plus faible qu'une série d'obstacles disposés en profondeur. Un attaquant qui franchit la ligne extérieure se heurte encore à une deuxième puis à une troisième, et chaque ligne franchie lui coûte du temps et des ressources pendant que les défenseurs se regroupent.

La pratique de la sécurité a repris la métaphore presque telle quelle. Le NIST définit la défense en profondeur comme une stratégie de sécurité de l'information qui intègre des capacités liées aux **personnes, à la technologie et aux opérations** afin d'établir des barrières variables sur plusieurs couches et missions de l'organisation. Cette définition en trois volets compte : les couches ne se limitent pas aux pare-feu et aux logiciels, elles incluent aussi un personnel formé, des procédures et le travail quotidien qui maintient les contrôles en état de marche. L'OWASP Developer Guide décrit le même objectif pour le logiciel : éliminer les points uniques de compromission totale en combinant plusieurs protections diversifiées.

## Un e-mail d'hameçonnage, cinq occasions de l'arrêter

Prenons un employé qui ouvre un e-mail d'hameçonnage convaincant et saisit son mot de passe sur une fausse page de connexion. Dans une conception à couche unique, c'est une compromission totale. Dans une conception en couches, l'attaquant doit encore franchir chacune des lignes ci-dessous.

| Couche | Ce qu'elle fait ici | Si cette couche échoue aussi |
|---|---|---|
| Identité et accès | L'authentification multifacteur bloque à elle seule le mot de passe volé ; le [moindre privilège](least-privilege.md) fait que le compte ne peut atteindre que ce que le poste exige | L'attaquant est à l'intérieur, mais limité au périmètre d'un seul utilisateur |
| Réseau | La segmentation et les pare-feu isolent la zone de cet utilisateur des serveurs et des systèmes sensibles | Le mouvement latéral devient possible |
| Hôte et application | Des systèmes corrigés et durcis ainsi que la validation des entrées empêchent les exploits faciles nécessaires à l'élévation de privilèges | L'attaquant peut exécuter du code sur davantage de machines |
| Données | Une autorisation au niveau de chaque enregistrement limite ce qu'un compte peut lire, et des sauvegardes hors ligne gardent l'information récupérable (le chiffrement au repos et en transit protège les disques volés et le trafic intercepté, pas les données lues via un compte valide) | Les données accessibles au compte peuvent être exposées, mais restent restaurables |
| Surveillance et réponse | La journalisation, la détection d'intrusion et la réponse aux incidents repèrent les connexions et déplacements inhabituels, puis les contiennent | L'intrusion passe inaperçue, ce qui est le véritable pire scénario |

En lisant le tableau de haut en bas, trois effets ressortent. Chaque couche **contient le rayon d'impact**, en réduisant ce que l'attaquant peut atteindre. Chacune **fait gagner du temps**, puisque chaque barrière implique plus de travail et plus de bruit. Et chacune **génère des signaux** que la couche de surveillance peut capter, ce qui explique pourquoi la journalisation et la [responsabilité](accountability.md) traversent toutes les autres couches plutôt que de se situer à la fin. L'erreur humaine du départ, tomber dans le piège, ne devient jamais à elle seule une compromission totale. Les contrôles physiques, comme des locaux verrouillés, forment une couche extérieure autour de l'ensemble.

## L'indépendance est tout l'enjeu

Deux couches qui échouent pour la même raison ne sont en réalité qu'une seule couche dessinée deux fois. C'est la manière la plus fréquente dont la défense en profondeur dérape :

- **Identifiants partagés :** si le VPN, la messagerie et la console du serveur acceptent tous le même mot de passe sans second facteur, un seul hameçonnage réussi ouvre les trois.
- **Défauts partagés :** deux filtres construits sur la même bibliothèque d'analyse syntaxique laisseront tous deux passer la même entrée malformée.
- **Hypothèses partagées :** si chaque système interne fait confiance à tout ce qui provient de l'intérieur du réseau, les couches internes disparaissent dès que le périmètre est franchi. Le [Zéro confiance](zero-trust.md) répond précisément à ce problème en vérifiant chaque requête au lieu de la supposer sûre.

Un test utile pour toute paire de contrôles : nommer un événement réaliste qui viendrait à bout des deux. Si c'est facile, ils sont redondants mais pas indépendants. La diversité des mécanismes (quelque chose que l'on sait plus quelque chose que l'on possède, des règles réseau plus des règles d'hôte, la prévention plus la détection) est ce qui permet réellement aux couches de s'additionner.

## Comment les couches disparaissent en silence

Même un empilement bien conçu se dégrade si personne ne l'entretient. Le schéma d'architecture montre souvent plus de défenses qu'il n'en existe réellement.

1. **Superposition pour cocher des cases.** Des outils sont achetés et déployés sans être intégrés ; ils produisent alors des alertes que personne ne consulte et laissent des lacunes dont personne n'est responsable.
2. **Excès de confiance dans le périmètre.** Le mur extérieur est jugé suffisant et l'intérieur reste fragile, de sorte qu'une seule brèche expose tout.
3. **Dérive opérationnelle.** Les configurations changent, les certificats expirent, les exceptions s'accumulent et une surveillance trop bruyante est réduite au silence ; chaque étape retire une couche sans que personne ne l'ait décidé.

Le remède est en partie technique et en partie organisationnel. La [sécurité dès la conception](security-by-design.md) planifie les couches dans un système dès le départ plutôt que de les ajouter après coup, et des revues régulières vérifient que chaque couche fonctionne toujours comme le prétend le schéma. Le NIST SP 800-53 y contribue en organisant ses contrôles en familles (contrôle d'accès, protection des systèmes et des communications, audit et responsabilité, réponse aux incidents, entre autres) qui décrivent ensemble les couches qu'une organisation est censée maintenir.

## Références

- [Glossaire du NIST : Defense in Depth](https://csrc.nist.gov/glossary/term/defense_in_depth)
- [NIST SP 800-53 Rev. 5, Security and Privacy Controls for Information Systems and Organizations](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [OWASP Developer Guide : Security Principles (Defense in Depth)](https://devguide.owasp.org/en/02-foundations/03-security-principles/)
- [CISA : Layering Network Security Through Segmentation](https://www.cisa.gov/resources-tools/resources/layering-network-security-through-segmentation-infographic)
