---
title: "Chapeaux blancs, noirs et gris : classer les hackers selon leur intention"
description: "Un cadre qui classe les praticiens de la sécurité selon leur intention et leur autorisation plutôt que selon leurs compétences techniques."
keywords:
  - chapeau blanc
  - chapeau noir
  - chapeau gris
  - hacking éthique
  - éthique du hacker
  - autorisation
---

# Chapeaux blancs, noirs et gris

**Les couleurs de chapeau** sont un raccourci pour classer les personnes qui s'introduisent dans les systèmes selon deux critères que les seules compétences techniques ne peuvent pas saisir : leur intention et leur autorisation. Un chapeau blanc, un chapeau noir et un chapeau gris peuvent utiliser exactement les mêmes outils et les mêmes techniques. Ce qui les distingue, c'est de savoir s'ils ont la permission d'agir, et s'ils cherchent à protéger ou à nuire. La métaphore est empruntée aux westerns de facture classique, où les héros portaient des chapeaux blancs et les méchants des chapeaux noirs.

## Deux questions qui décident de la couleur

Toute classification se ramène à deux questions posées sur la même activité :

1. **Le propriétaire du système l'a-t-il autorisée ?** Un contrat, un périmètre défini, une politique de divulgation publiée ou les conditions d'un programme de bug bounty valent tous permission. La curiosité, les bonnes intentions ou le fait qu'un système soit facile d'accès ne la valent pas.
2. **Quel était le but de l'accès ?** Trouver et corriger des faiblesses va dans un sens ; le vol, la perturbation, l'extorsion ou le gain personnel vont dans l'autre.

Le croisement des deux réponses donne les catégories bien connues :

| | Intention protectrice | Intention nuisible ou intéressée |
| --- | --- | --- |
| **Autorisé** | Chapeau blanc : testeurs d'intrusion, chercheurs en sécurité et défenseurs internes travaillant dans le cadre d'un contrat, d'un périmètre ou d'une politique de divulgation | Rare en théorie, mais bien réel : un employé interne ou un prestataire qui abuse d'un accès légitime |
| **Non autorisé** | Chapeau gris : sonde un système sans permission, puis signale la faille au lieu de l'exploiter | Chapeau noir : intrusion à des fins de gain, de perturbation ou d'autres préjudices, ce que la plupart des gens appellent la "cybercriminalité" |

La leçon que le tableau rend visible est que c'est l'autorisation, et non la capacité, qui constitue la ligne de partage. Le même scan qui est un service payant dans un contexte est un délit dans un autre, et c'est pourquoi la question de l'[autorisation et du consentement](../ethics-and-philosophy/authorization-and-consent.md) compte davantage que n'importe quel outil. C'est aussi une illustration directe des [connaissances à double usage](../ethics-and-philosophy/dual-use-knowledge.md) : la compétence elle-même n'a pas de couleur.

## Du "hacker" aux chapeaux : la naissance des étiquettes

Le langage des chapeaux est né de la longue bataille de la communauté des hackers sur le sens que devait avoir le mot "hacker". Dans les premières décennies de l'informatique, un hacker était simplement un programmeur compétent et curieux, l'identité retracée dans [Origines](origins.md) et célébrée dans le [Manifeste du hacker](hacker-manifesto.md). À mesure que les intrusions et la criminalité informatique sont apparues aux yeux du public au cours des années 1980 et 1990, la presse a de plus en plus employé "hacker" au sens de "criminel", et de nombreux praticiens ont voulu un moyen de distinguer le travail de sécurité honnête de l'intrusion malveillante.

Les étiquettes de couleur offraient cette distinction, en reprenant le code vestimentaire des westerns. Leur sens a toutefois évolué. Le Jargon File mentionne "gray hat" comme un terme occasionnel désignant des personnes ayant des compétences de cracker mais agissant dans le cadre de la loi, par exemple lors d'évaluations de sécurité, ce qui se rapproche du chapeau blanc actuel. Le sens moderne de chapeau gris, une personne qui agit sans permission mais avec de bonnes intentions, est apparu plus tard. Lorsque les tests d'intrusion commerciaux et les programmes de bug bounty ont mûri dans les années 2000, "chapeau blanc" était devenu une identité professionnelle reconnue plutôt qu'un simple terme d'argot.

## Les cas qui échappent à une seule couleur

Le modèle à deux questions est une simplification, et les cas difficiles montrent où il atteint ses limites :

- **Le signalement de faille non sollicité.** Un chercheur remarque une faille sur un site web public, la confirme sans permission et écrit au propriétaire. L'intention est protectrice, mais l'accès n'était pas autorisé : l'acte se situe donc en pleine zone grise, et ses conséquences juridiques dépendent de la juridiction et de la réaction du propriétaire.
- **Le testeur qui sort du périmètre.** Un testeur d'intrusion mandaté suit une piste prometteuse jusqu'à un système que le contrat ne couvrait pas. La personne est un chapeau blanc de métier, et pourtant cet accès précis n'était pas autorisé.
- **La "recherche" utilisée comme moyen de pression.** Quelqu'un découvre une vulnérabilité et demande à être payé en échange de son silence. Parler de recherche ne change pas la couleur : le département de la Justice des États-Unis cite explicitement la découverte de vulnérabilités dans le but d'extorquer leurs propriétaires comme un exemple d'activité qui n'est pas de bonne foi.
- **L'employé autorisé qui cause un préjudice.** La permission seule ne fait pas d'un acte celui d'un chapeau blanc. Un employé qui utilise un accès légitime pour voler des données dispose d'une autorisation, mais son intention est nuisible.

Ces cas expliquent pourquoi la communauté traite les couleurs comme un vocabulaire de discussion plutôt que comme un verdict juridique. Les normes qui leur donnent du poids sont approfondies dans [Sous-cultures et valeurs](subcultures-and-values.md).

## Les voies qui transforment le gris en blanc

Une grande partie de la pratique moderne de la sécurité existe pour offrir aux personnes compétentes et curieuses une voie légale, la même pulsion que celle décrite dans [Curiosité et innovation](curiosity-and-innovation.md) :

- **La divulgation coordonnée des vulnérabilités.** Des programmes comme le processus géré par la CISA offrent aux chercheurs un canal défini pour signaler les failles et une séquence prévisible pour les correctifs et la publication.
- **Les bug bounties et les clauses de safe harbor.** En publiant un périmètre et en s'engageant à ne pas poursuivre les personnes qui signalent de bonne foi, une organisation transforme ce qui aurait été un sondage de chapeau gris en travail autorisé.
- **La politique de poursuites.** En mai 2022, le département de la Justice des États-Unis a annoncé que la recherche en sécurité de bonne foi ne devait pas faire l'objet de poursuites au titre du Computer Fraud and Abuse Act, en la définissant comme un accès effectué uniquement pour tester, examiner ou corriger une faille de sécurité, d'une manière conçue pour éviter tout préjudice.
- **Les titres professionnels.** "Hacker éthique" et "chercheur en sécurité" sont désormais des intitulés de poste courants, bâtis directement sur l'idée du chapeau blanc, et le NIST décrit les tests d'intrusion comme des tests qui reproduisent des attaques réelles dans des conditions définies.

De nombreux litiges juridiques dépendent encore de la question de savoir si l'accès était autorisé, ce qui correspond exactement à l'écart que désigne l'étiquette de chapeau gris. Chacun de ces mécanismes réduit cet écart en rendant la permission explicite avant que quiconque ne commence à sonder.

## Références

- [EFF : Coders' Rights Project](https://www.eff.org/issues/coders)
- [Glossaire du NIST : Penetration Testing](https://csrc.nist.gov/glossary/term/penetration_testing)
- [CISA : Coordinated Vulnerability Disclosure Process](https://www.cisa.gov/coordinated-vulnerability-disclosure-process)
- [Département de la Justice des États-Unis : New Policy for Charging Cases under the Computer Fraud and Abuse Act (2022)](https://www.justice.gov/opa/pr/department-justice-announces-new-policy-charging-cases-under-computer-fraud-and-abuse-act)
- [The Jargon File : entrée "black hat"](http://www.catb.org/jargon/html/B/black-hat.html)
- [The Jargon File : entrée "hacker"](http://www.catb.org/jargon/html/H/hacker.html)
