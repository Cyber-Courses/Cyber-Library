---
title: "Menaces et acteurs de la menace : qui attaque et pourquoi"
description: "Ce qu'est une menace, les types d'acteurs qui se cachent derrière, et les motivations et capacités qui rendent une menace réelle plutôt qu'hypothétique."
keywords:
  - menace
  - acteur de la menace
  - modélisation des menaces
  - motivation de l'adversaire
  - capacité de l'attaquant
  - renseignement sur les cybermenaces
---

# Menaces et acteurs de la menace

**Une menace est toute circonstance ou tout événement susceptible de nuire à un système**, et un acteur de la menace est la personne ou le groupe qui la concrétiserait. Le travail de sécurité commence ici, car on ne peut pas se défendre contre un danger que l'on n'a pas nommé. Une menace n'est ni la faiblesse elle-même ni le dommage lui-même ; c'est la possibilité qui se situe entre les deux, attendant quelqu'un qui aurait une raison d'agir.

## Une menace n'est ni une faiblesse ni une perte

Ces trois notions sont faciles à confondre, il est donc utile de les distinguer :

- Une **menace** est le potentiel de dommage, par exemple "un attaquant pourrait chiffrer nos fichiers contre une rançon".
- Une [vulnérabilité](vulnerabilities-and-exploits.md) est la faille qui lui permettrait de réussir, comme un serveur non corrigé.
- Le **dommage** est la perte réalisée, mesurée à l'aune de la [triade CIA](cia-triad.md) : des données divulguées, altérées ou rendues indisponibles.

Une menace sans faiblesse correspondante reste hypothétique. Une faiblesse qu'aucun acteur ne se soucie d'exploiter reste dormante. Pour une attaque délibérée, le risque n'apparaît que lorsque les deux coïncident sur un [actif](assets-and-attack-surface.md) qui a de la valeur. Les menaces non adverses brisent ce schéma : une coupure de courant, un incendie, une panne de disque ou une suppression accidentelle peuvent nuire au même actif sans aucun attaquant ni aucune faiblesse exploitée, et c'est pourquoi les évaluations de risque pèsent les accidents et l'environnement au même titre que les acteurs hostiles.

## Les acteurs, classés selon ce qui les motive

Celui qui se cache derrière une menace détermine tout de son comportement : à quel point il est patient, combien il est prêt à dépenser et ce qu'il est prêt à briser. Le renseignement sur la menace regroupe les acteurs moins selon leurs outils que selon leur motif.

| Acteur | Ce qu'ils veulent | Capacité typique |
| --- | --- | --- |
| Cybercriminels | De l'argent, par la fraude, le vol ou la rançon | Modérée à élevée ; achètent souvent outils et accès |
| Groupes étatiques | Espionnage, sabotage, avantage stratégique | Très élevée ; patients, bien financés, outillage sur mesure |
| Hacktivistes | De l'attention pour une cause | Faible à modérée ; privilégient la perturbation et les fuites |
| Initiés malveillants | Rancune ou profit | Variable, mais partent d'un accès légitime |
| Opportunistes | Tout ce qui est facile, y compris la vantardise | Faible ; s'appuient sur des failles connues et l'automatisation |

L'initié négligent rappelle que tout acteur n'est pas hostile. Un employé qui envoie par courriel un tableur à la mauvaise adresse reste une source de dommage dont vos défenses doivent tenir compte.

## Capacité, intention et opportunité

Pour qu'une menace soit crédible, trois conditions doivent être réunies en même temps. Les analystes appellent parfois cela le triangle de la menace :

1. **Capacité.** L'acteur dispose des compétences, des outils ou des ressources pour mener l'attaque à bien.
2. **Intention.** L'acteur a une raison de vous cibler vous plutôt que quelqu'un d'autre.
3. **Opportunité.** Une faiblesse accessible existe, qu'il peut exploiter.

Les défenseurs modifient rarement la capacité ou l'intention d'un adversaire. Ce qu'ils peuvent réduire, c'est l'opportunité, et c'est pourquoi une grande partie de la sécurité consiste à fermer les ouvertures avant que quiconque ayant un motif ne les trouve.

## Suivre un groupe de ransomware

Observez comment les pièces s'assemblent dans un schéma courant :

1. **Motif.** Un groupe criminel veut un paiement, de sorte que toute organisation qui ne peut pas se permettre une interruption de service devient une cible.
2. **Entrée.** Ils piègent un employé par phishing, exploitant l'opportunité la moins coûteuse plutôt que la faille la plus sophistiquée.
3. **Propagation.** Avec un seul point d'appui, ils progressent dans le réseau vers des données de valeur.
4. **Levier.** Dans les attaques modernes à double extorsion, ils exfiltrent d'abord les données, brisant la confidentialité, puis chiffrent les systèmes pour briser la disponibilité, de sorte qu'une victime disposant de bonnes sauvegardes reste confrontée à la menace d'une fuite.

À chaque étape, un [contrôle](controls-and-countermeasures.md) aurait pu supprimer l'opportunité : une sensibilisation contre le phishing, le [moindre privilège](../principles/least-privilege.md) pour limiter la propagation, et des sauvegardes hors ligne pour atténuer la rançon. Le motif de l'acteur n'a jamais changé ; les défenseurs ne lui ont simplement laissé nulle part où atterrir.

## Références

- [NIST SP 800-30 Rev. 1, Guide for Conducting Risk Assessments](https://csrc.nist.gov/pubs/sp/800/30/r1/final)
- [MITRE ATT&CK Groups](https://attack.mitre.org/groups/)
- [CISA, Cyber Threats and Advisories](https://www.cisa.gov/topics/cyber-threats-and-advisories)
- [ENISA Threat Landscape](https://www.enisa.europa.eu/topics/cyber-threats/threats-and-trends)
- [Verizon Data Breach Investigations Report (DBIR)](https://www.verizon.com/business/resources/reports/dbir/)
