---
title: "La triade CIA : les trois propriétés d'un système sécurisé"
description: "La confidentialité, l'intégrité et la disponibilité sont les trois propriétés qui, ensemble, définissent ce que signifie la sécurité d'un système et de ses données."
keywords:
  - triade cia
  - confidentialité
  - intégrité
  - disponibilité
  - propriétés de sécurité
  - triade dad
---

# La triade CIA

**La triade CIA** décompose l'objectif vague de la "sécurité" en trois propriétés concrètes qui méritent d'être protégées : la confidentialité, l'intégrité et la disponibilité. Son intérêt est de donner à chaque contrôle, risque et incident une étiquette précise. Au lieu de dire qu'un système a été "piraté", vous pouvez indiquer laquelle des trois propriétés a échoué, et cette seule distinction détermine généralement la manière de réagir.

## La confidentialité, l'intégrité et la disponibilité, une par une

Chaque lettre répond à une question différente sur une donnée ou un service. Une propriété n'a d'importance que parce qu'un "non" à sa question causerait un préjudice réel aux [actifs](assets-and-attack-surface.md) dont vous dépendez.

| Propriété | La question à laquelle elle répond | Assurée par |
| --- | --- | --- |
| Confidentialité | Seules les personnes autorisées peuvent-elles la lire ? | Chiffrement, contrôle d'accès, authentification |
| Intégrité | Est-elle exacte, et remarquerions-nous une modification ? | Codes d'authentification de message (MAC), signatures numériques, journaux de modifications |
| Disponibilité | Les bonnes personnes peuvent-elles y accéder au moment voulu ? | Redondance, sauvegardes, planification de capacité |

Les trois sont indépendantes : un système peut être parfaitement confidentiel tout en étant constamment hors ligne, ou hautement disponible tout en étant rempli d'enregistrements silencieusement altérés.

## La triade DAD : comment chaque propriété échoue

Chaque propriété a une défaillance correspondante, et ensemble elles forment ce que l'on appelle parfois la triade DAD. Nommer la défaillance vous oriente vers les [contrôles](controls-and-countermeasures.md) qui la préviennent.

- **La divulgation** (Disclosure) brise la confidentialité. Des identifiants volés, un espace de stockage public, ou un trafic transmis en clair. La donnée est inchangée et toujours présente, mais quelqu'un qui ne devrait pas la voir la voit désormais, et cela ne peut pas être annulé.
- **L'altération** (Alteration) brise l'intégrité. Un attaquant modifie un enregistrement, un logiciel malveillant patche un binaire, ou un bogue corrompt une table. Le vrai danger est la version que personne ne remarque.
- **La destruction** (ou le déni) brise la disponibilité. Un chiffrement par ransomware, un déluge de trafic, une sauvegarde supprimée, ou une coupure de courant. Une donnée que personne ne peut atteindre n'a aucune valeur pour les personnes qui en ont besoin. Les ransomwares modernes brisent aussi souvent la confidentialité, en exfiltrant les données avant de les chiffrer afin que les opérateurs puissent menacer de les divulguer.

La plupart des [vulnérabilités](vulnerabilities-and-exploits.md) sont notées selon laquelle de ces trois propriétés elles permettent à un attaquant d'atteindre, et avec quelle gravité.

## Un serveur de build, trois façons d'en abuser

Imaginez un attaquant qui obtient l'accès au serveur de build logiciel d'une entreprise. Un seul point d'ancrage menace les trois propriétés à la fois :

1. **Contre la confidentialité,** il copie le code source et les clés de signature stockées à côté.
2. **Contre l'intégrité,** il glisse une backdoor dans la prochaine version, qui est livrée à chaque client en portant une signature valide.
3. **Contre la disponibilité,** il efface le pipeline en partant, stoppant tous les builds futurs.

La même intrusion se lit comme une fuite, une falsification et une panne selon la propriété que vous examinez. C'est pourquoi "nous avons reconstruit le serveur" signifie rarement que l'incident est terminé.

## Quand les trois propriétés s'opposent

Les objectifs entrent régulièrement en conflit, et la triade rend ces compromis explicites plutôt qu'accidentels. Un chiffrement fort augmente la confidentialité mais peut détruire la disponibilité si une clé est perdue. Des copies de sauvegarde supplémentaires augmentent la disponibilité mais multiplient les endroits où des données confidentielles peuvent fuir. Il n'y a pas de classement figé : les dossiers médicaux accordent le plus de poids à la confidentialité et à l'intégrité, tandis qu'une page de statut publique vit ou meurt selon sa disponibilité. Superposer les protections, comme dans la [défense en profondeur](../principles/defense-in-depth.md), est la manière dont les équipes servent plus d'une propriété sans tout miser sur une seule mesure.

## Ce que les trois lettres laissent de côté

La triade est une fondation, pas un modèle complet. Elle dit peu de chose sur l'**authenticité** (cette donnée ou cet utilisateur est-il authentique ?), la **non-répudiation** (quelqu'un peut-il ensuite nier une action ?), ou la [responsabilité](../principles/accountability.md) (une action peut-elle être attribuée à celui qui l'a entreprise ?). Certaines normes intègrent ces notions dans l'intégrité ; d'autres les listent séparément. Considérez la CIA comme le premier vocabulaire auquel vous recourez, pas le dernier.

## Références

- [NIST SP 800-12 Rev. 1, An Introduction to Information Security](https://csrc.nist.gov/pubs/sp/800/12/r1/final)
- [FIPS 199, Standards for Security Categorization of Federal Information and Information Systems](https://csrc.nist.gov/pubs/fips/199/final)
- [NIST Glossary: Confidentiality](https://csrc.nist.gov/glossary/term/confidentiality)
- [ISO/IEC 27000, Information security management systems overview and vocabulary](https://www.iso.org/standard/73906.html)
