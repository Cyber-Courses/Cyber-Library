---
title: "Défense en profondeur : des contrôles en couches pour qu'aucune défaillance isolée n'expose tout"
description: "Le principe de sécurité consistant à empiler des contrôles indépendants et superposés afin que la défaillance de l'un d'eux n'expose pas l'ensemble du système."
keywords:
  - défense en profondeur
  - sécurité en couches
  - contrôles superposés
  - contrôles indépendants
  - architecture de sécurité
  - protections redondantes
---

# Défense en profondeur

La **défense en profondeur** est la pratique consistant à protéger un système par plusieurs couches indépendantes de contrôles, de sorte qu'un attaquant qui déjoue une couche en rencontre encore une autre, et qu'aucune défaillance isolée n'expose tout. L'hypothèse qui la sous-tend est humble mais réaliste : chaque contrôle individuel finira par défaillir, être contourné ou être mal configuré, c'est pourquoi la sécurité ne devrait jamais reposer sur la perfection d'une seule protection.

## De la forteresse au pare-feu

L'idée est plus ancienne que l'informatique. Les forteresses médiévales combinaient des douves, un mur d'enceinte, un donjon intérieur et des défenseurs armés, de sorte que franchir le premier obstacle ne donnait à l'attaquant qu'une vue du suivant. La doctrine militaire a ensuite employé l'expression "défense en profondeur" pour décrire le fait de céder lentement du terrain sur des positions préparées plutôt que de tout miser sur une seule ligne de front.

La sécurité de l'information a emprunté le terme directement. La National Security Agency des États-Unis a promu une stratégie de "défense en profondeur" qui organisait la protection autour des personnes, de la technologie et des opérations plutôt qu'autour d'un unique équipement de périmètre. À mesure que les réseaux d'entreprise à plat ont cédé la place aux services cloud et au travail à distance, la leçon s'est affinée : une coquille extérieure dure avec un intérieur mou échoue gravement dès que quelque chose pénètre à l'intérieur.

## Des couches qui doivent défaillir indépendamment

La puissance de la mise en couches vient de l'indépendance. Deux contrôles qui partagent le même défaut, le même identifiant ou le même fournisseur peuvent tomber ensemble, ce qui n'est en réalité qu'une seule couche portant deux manteaux. Une défense efficace répartit les contrôles sur différentes catégories afin qu'une faiblesse dans l'une ne se propage pas aux autres.

| Couche | Exemple de contrôle | Ce qu'elle apporte |
|---|---|---|
| Physique | Baies verrouillées, accès par badge | Empêche l'altération directe du matériel |
| Réseau | Segmentation, pare-feu | Limite la propagation d'un point d'ancrage |
| Hôte | Application de correctifs, protection des terminaux | Durcit chaque machine isolément |
| Application | Validation des entrées, revue de code | Bloque l'abus du logiciel lui-même |
| Données | Chiffrement, sauvegardes | Garde les données utiles même copiées ou détruites |
| Personnes | Formation, étapes d'approbation | Rattrape ce que la technologie manque |

## Accompagner un intrus à travers les couches

Prenons un e-mail de phishing qui pousse un employé à saisir son mot de passe sur une page falsifiée.

1. **La première couche ne retient rien.** Le mot de passe est volé, donc l'authentification seule a échoué.
2. **Un second facteur bloque la connexion.** L'authentification multifacteur empêche le mot de passe volé de fonctionner à lui seul.
3. **La segmentation contient l'explosion.** Si l'attaquant parvient tout de même à entrer, les frontières réseau le tiennent à l'écart des systèmes sans rapport.
4. **La surveillance donne l'alerte.** Les accès inhabituels sont journalisés et signalés, de sorte que les intervenants peuvent agir avant que les dégâts ne s'étendent.

Aucune couche n'était parfaite, pourtant le système a survécu parce que la couche suivante était prête.

## La séduction d'un faux plancher

La profondeur peut tromper. Empiler de nombreux contrôles crée un sentiment réconfortant de sécurité qui est faux lorsque les couches dépendent discrètement de la même chose, comme un seul fournisseur d'identité ou un seul compte d'administrateur. Plus de couches signifie aussi plus de coûts, plus de complexité et davantage à maintenir, et un contrôle négligé est une brèche déguisée en mur. L'objectif est une couverture indépendante des risques réels, et non un théâtre de la sécurité mesuré au nombre de cases cochées.

## Comment les couches sont construites

La défense en profondeur est un cadre, et les autres principes en fournissent la matière. Le [moindre privilège](least-privilege.md) et le [zero trust](zero-trust.md) maintiennent chaque couche petite et vérifiée afin qu'une brèche de l'une reste contenue, tandis que la [sécurité dès la conception](security-by-design.md) garantit que les couches existent avant un incident plutôt qu'après.

## Références

- [NIST SP 800-53 Rev. 5, Security and Privacy Controls](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [NIST SP 800-160 Vol. 2 Rev. 1, Developing Cyber-Resilient Systems](https://csrc.nist.gov/pubs/sp/800/160/v2/r1/final)
- [CISA, Layering Network Security Through Segmentation](https://www.cisa.gov/news-events/news/layering-network-security-through-segmentation)
- [OWASP, Defense in Depth](https://owasp.org/www-community/Defense_in_Depth)
