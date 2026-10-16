---
title: "La responsabilité morale : le devoir qui accompagne le pouvoir sur les systèmes"
description: "La responsabilité morale est le devoir que les praticiens ont envers les utilisateurs, les organisations et le public lorsqu'ils détiennent un pouvoir sur des systèmes que d'autres ne peuvent ni voir, ni inspecter, ni consentir."
keywords:
  - responsabilité morale
  - éthique professionnelle
  - devoir de diligence
  - bien public
  - code d'éthique de l'acm
  - éthique de la sécurité
---

# La responsabilité morale

**La responsabilité morale est le devoir qu'un praticien porte parce que ses décisions atteignent des personnes qui n'y ont jamais consenti et qui souvent n'en ont jamais connaissance.** Un ingénieur sécurité qui affaiblit une valeur par défaut, un chercheur qui publie un exploit, ou un administrateur qui retarde un correctif agit au nom des utilisateurs, d'un employeur et d'un public plus large qui n'est pas dans la pièce. Détenir un pouvoir sur un système n'est pas neutre. Cela crée des obligations qui survivent à n'importe quelle tâche isolée, et cela rend le praticien comptable des conséquences qu'il déclenche, même lorsqu'aucune règle n'a été enfreinte.

## L'asymétrie qui crée le devoir

Les transactions ordinaires supposent une égalité approximative : les deux parties peuvent inspecter ce à quoi elles s'engagent. Le travail de sécurité brise cette hypothèse. Les personnes concernées par un choix de conception ne peuvent généralement pas lire le code, ne peuvent pas voir les journaux, et ne peuvent pas juger si un risque a été bien géré. Elles dépendent de quelqu'un qu'elles ne rencontreront jamais.

Ce pouvoir à sens unique est la raison pour laquelle la compétence seule ne suffit pas. Là où un utilisateur ne peut pas se protéger lui-même, la personne qui le peut a le devoir d'agir dans son intérêt, et non simplement dans la limite de ses droits. Avoir la permission d'agir, l'objet de [l'autorisation et le consentement](authorization-and-consent.md), détermine si un acte est autorisé ; cela ne détermine pas s'il était juste.

## Répondre avant et après les faits

Les philosophes divisent la responsabilité en deux directions, et les deux lient le praticien à la fois.

| Direction | La question qu'elle pose | Ce à quoi cela ressemble en pratique |
|---|---|---|
| Tournée vers l'avenir | Que suis-je obligé de faire ou d'empêcher ? | Choisir une valeur par défaut sûre, divulguer une faille, refuser de livrer une faiblesse connue |
| Tournée vers le passé | Qui doit répondre de ce qui s'est déjà produit ? | Assumer une violation, expliquer une décision, accepter les retombées d'un raccourci |

Le devoir tourné vers l'avenir est le plus lourd en sécurité, car la plupart des préjudices sont évitables avant de survenir. Un praticien ne peut pas attendre un incident pour découvrir qu'il était responsable depuis le début.

## Ce que le code de l'ACM place en premier

Les codes professionnels transforment ce devoir abstrait en un ordre hiérarchisé. Le Code d'éthique de l'ACM s'ouvre en demandant aux professionnels de l'informatique de "contribuer à la société et au bien-être humain" et d'"éviter les préjudices", et il affirme que le bien public est la préoccupation centrale de tout travail informatique professionnel. Les codes de l'IEEE et de l'(ISC)2 font écho à la même priorité : lorsque les intérêts d'un employeur, d'un client et du public entrent en collision, le public passe en premier.

C'est ici que se rencontrent l'intention, la conséquence et les valeurs concurrentes. Les bonnes intentions ne libèrent pas du devoir si le résultat prévisible est un préjudice, et une instruction licite de la part d'un employeur ne l'emporte pas sur l'obligation envers les personnes en aval.

## Une faille que vous êtes le seul à connaître

Considérez comment le devoir mord dans un moment unique et courant. Vous trouvez une vulnérabilité grave dans un produit sur lequel des millions de personnes comptent.

1. **La capacité est neutre.** La même découverte peut être discrètement transformée en arme ou signalée de manière responsable. La connaissance est une [connaissance à double usage](dual-use-knowledge.md) ; ce que vous devez est décidé par ce que vous faites ensuite.
2. **Le devoir tourné vers l'avenir s'active.** Vous êtes désormais l'un des rares à pouvoir empêcher un préjudice, ce qui signifie que le silence est lui-même un choix aux conséquences.
3. **La redevabilité suit le choix.** Que vous coordonniez la divulgation, vendiez la faille ou la publiiez publiquement, vous devrez répondre de qui a été blessé et de qui a été protégé.

Aucune règle externe n'impose l'étape deux ou trois. La force provient de la position dans laquelle vous vous trouvez, et non d'une politique.

## Là où la responsabilité devient redevabilité

La responsabilité morale est le devoir lui-même ; la [redevabilité](../principles/accountability.md) est la machinerie, telle que les journaux, la propriété et la revue, qui permet de retracer une décision jusqu'à une personne et de la faire répondre. La responsabilité est aussi ce qui demeure lorsque les réponses faciles s'épuisent : lorsque protéger un groupe en expose un autre, le praticien ne peut échapper à la pesée, une tension explorée dans [sécurité contre vie privée](security-versus-privacy.md). Le devoir ne disparaît pas parce qu'un choix est difficile ; ce sont précisément les choix difficiles qu'il existe pour gouverner.

## Références

- [ACM Code of Ethics and Professional Conduct](https://www.acm.org/code-of-ethics)
- [IEEE Code of Ethics](https://www.ieee.org/about/corporate/governance/p7-8.html)
- [(ISC)2 Code of Ethics](https://www.isc2.org/ethics)
- [Stanford Encyclopedia of Philosophy: Moral Responsibility](https://plato.stanford.edu/entries/moral-responsibility/)
