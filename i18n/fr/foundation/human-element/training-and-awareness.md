---
title: "Formation et sensibilisation : combler le manque de connaissances humaines"
description: "Le manque de connaissances qui empêche les personnes de reconnaître les menaces, et la formation continue qui les aide à repérer les attaques et à y répondre."
keywords:
  - sensibilisation à la sécurité
  - formation à la sécurité
  - sensibilisation au phishing
  - erreur humaine
  - culture de sécurité
  - ingénierie sociale
---

# Formation et sensibilisation

**La formation et la sensibilisation** désignent l'effort continu visant à donner aux personnes les connaissances et les habitudes dont elles ont besoin pour reconnaître les menaces de sécurité et y répondre. La plupart des contrôles techniques dépendent encore d'un humain qui fait un choix sûr. Lorsque cette connaissance fait défaut, les attaquants n'ont pas besoin de vaincre la technologie ; il leur suffit de demander à une personne de les laisser entrer. L'écart entre ce que les personnes sont censées gérer et ce qu'on leur a réellement enseigné est l'un des facteurs les plus courants des incidents de sécurité.

## Les décisions d'une fraction de seconde que la technologie confie aux personnes

La formation est plus facile à concevoir lorsqu'on part des moments où c'est une personne, et non un système, qui décide de l'issue. Le phishing et les autres attaques d'ingénierie sociale visent précisément ces moments, car tromper une personne est souvent plus facile que de percer un système bien défendu.

- **Cliquer ou ignorer.** Un lien ou une pièce jointe arrive, d'apparence anodine, mais qui ne l'est pas.
- **Approuver ou refuser.** Une invite de connexion apparaît sur un téléphone alors que l'utilisateur ne l'a pas déclenchée.
- **Partager ou refuser.** Un interlocuteur prétendant appeler de la part du service informatique demande un mot de passe ou un code à usage unique.
- **Payer ou vérifier.** Une demande "urgente" invite le service financier à modifier les coordonnées bancaires d'un fournisseur.
- **Signaler ou se taire.** Quelque chose semble anormal, et la personne doit décider si cela mérite d'être remonté.

Chacune de ces situations est une décision de jugement qu'aucun filtre ne supprime totalement. Ce que les personnes savent à ce moment-là, et le fait qu'elles se sentent en sécurité pour agir face au doute, voilà ce que la formation a vocation à façonner.

## Sensibilisation, formation et compétences propres au rôle ne sont pas la même chose

Les programmes considèrent souvent que la "sensibilisation" constitue tout le travail. Des référentiels comme NIST SP 800-50 Rev. 1 et la famille de contrôles Awareness and Training (AT) de NIST SP 800-53 distinguent la sensibilisation générale destinée à tous d'une formation plus approfondie, fondée sur le rôle, pour les personnes dont le poste comporte des risques spécifiques.

| Niveau | Public visé | Résultat attendu | Exemple |
| --- | --- | --- | --- |
| Sensibilisation | Tout le monde, dès le premier jour | La reconnaissance : "ceci semble anormal" | Repérer un expéditeur usurpé ou une fausse page de connexion |
| Formation | Le personnel d'un rôle donné | Une réponse correcte et exercée | Le service financier vérifie les changements de coordonnées bancaires par téléphone, via un numéro connu |
| Compétences propres au rôle | Concepteurs et administrateurs | Un travail sûr par défaut | Les développeurs gardent les secrets hors du code ; les administrateurs appliquent des configurations de référence sûres |

Un contenu générique qui ignore les deux dernières lignes laisse le personnel financier sans rien entendre sur la fraude aux factures et les développeurs sans rien entendre sur les secrets dans le code. Les niveaux les plus approfondis sont directement liés aux [erreurs de conception et de développement](design-and-development-errors.md) et à la [mauvaise configuration](misconfiguration.md), car nombre de ces erreurs remontent à des personnes à qui l'on n'a jamais enseigné des valeurs par défaut sûres.

## Cinq façons dont un programme échoue discrètement

1. **L'accès avant l'explication.** Les nouveaux collaborateurs reçoivent leurs identifiants avant que quiconque n'aborde l'hygiène des mots de passe ou la façon de signaler un incident.
2. **La case à cocher annuelle.** Un unique module annuel est vite oublié et reflète rarement les techniques d'attaque actuelles.
3. **Un seul discours pour tous les rôles.** Le contenu ignore les menaces spécifiques auxquelles chaque équipe est réellement confrontée.
4. **La peur de la faute.** Les personnes qui s'attendent à une sanction cachent leurs erreurs, de sorte qu'un lien cliqué reste non signalé jusqu'à ce que les dégâts se propagent.
5. **Aucune mise en pratique.** Les leçons ne sont jamais répétées, si bien que les connaissances s'estompent bien avant d'être nécessaires.

Un autre échec, plus subtil, tient aux règles inexpliquées. Un utilisateur qui ne comprend pas pourquoi une règle existe est plus enclin à la contourner, affaiblissant discrètement les contrôles qui l'entourent. Cette dérive est étudiée dans [comportement et usage abusif des utilisateurs](user-behavior-and-misuse.md).

## Entretenir les connaissances entre deux incidents

Les programmes efficaces abordent l'apprentissage comme un cycle plutôt que comme un événement :

- **Commencer dès l'intégration, puis rafraîchir par sessions courtes.** Des sessions brèves et régulières valent mieux qu'un long cours annuel.
- **Adapter les exemples à chaque équipe.** Utiliser les attaques que cette équipe est susceptible de rencontrer, formulées dans son propre vocabulaire.
- **S'exercer avec des simulations.** Des exercices de phishing contrôlés et des exercices sur table développent une capacité de reconnaissance que la lecture passive ne peut pas apporter.
- **Faire du signalement un réflexe en un clic, sans culpabilisation.** Donner aux personnes un moyen simple et bien connu de faire remonter leurs préoccupations, et considérer chaque signalement comme une réussite. Des agences comme la CISA et le NCSC britannique publient des recommandations publiques pour reconnaître et signaler le phishing, qui peuvent alimenter ces contenus.
- **Laisser les résultats orienter le cycle suivant.** Concentrer les prochaines sessions sur les sujets et les équipes où la compréhension est la plus faible.

À terme, l'objectif est une culture où la sécurité est une habitude partagée plutôt qu'une case à cocher, afin que les bonnes décisions surviennent même dans des situations qu'aucune formation n'avait anticipées.

## Des signalements, pas seulement des clics : mesurer l'efficacité

Les taux de clic lors des simulations sont l'indicateur le plus visible, mais pris isolément ils peuvent induire en erreur : un faible taux de clic peut simplement signifier que le test était facile. Les taux de signalement comptent tout autant, car une personne qui reconnaît un message suspect et le signale devient un capteur, offrant aux défenseurs une alerte précoce que les outils seuls peuvent manquer. Suivre combien de personnes signalent, à quelle vitesse, et si les signalements continuent d'augmenter après une simulation montre si la culture évolue, et pas seulement si les personnes ont mémorisé une leçon.

La formation n'atteint jamais 100 %, c'est pourquoi elle va de pair avec des protections architecturales. Des approches comme le [zéro confiance](../principles/zero-trust.md) et le [moindre privilège](../principles/least-privilege.md) limitent ce qu'une seule décision mal informée peut exposer, de sorte qu'un personnel formé et une conception tolérante aux erreurs se renforcent mutuellement.

## Références

- [NIST SP 800-50 Rev. 1, Building a Cybersecurity and Privacy Learning Program](https://csrc.nist.gov/pubs/sp/800/50/r1/final)
- [NIST SP 800-53 Rev. 5, Security and Privacy Controls (AT, Awareness and Training)](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [CISA : Recognize and Report Phishing](https://www.cisa.gov/secure-our-world/recognize-and-report-phishing)
- [NCSC britannique : Phishing scams, how to spot and report them](https://www.ncsc.gov.uk/collection/phishing-scams)
- [SANS Security Awareness](https://www.sans.org/security-awareness-training/)
- [ENISA Threat Landscape](https://www.enisa.europa.eu/topics/cyber-threats/threats-and-trends)
