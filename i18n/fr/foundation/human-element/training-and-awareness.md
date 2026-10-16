---
title: "Formation et sensibilisation : combler le déficit de connaissances sur lequel comptent les attaquants"
description: "Le déficit de connaissances qui empêche les personnes de reconnaître les menaces, et l'éducation par couches qui transforme le personnel ordinaire en défenseurs vigilants."
keywords:
  - sensibilisation à la sécurité
  - formation à la sécurité
  - continuum d'apprentissage
  - reconnaissance du phishing
  - déficit de connaissances humain
  - culture du signalement
---

# Formation et sensibilisation

La **formation et la sensibilisation** sont la manière dont une organisation comble l'écart entre ce que les personnes sont censées gérer et ce qu'on leur a réellement appris. La plupart des outils de sécurité confient encore la décision finale à un humain : cliquer ou non, approuver ou refuser, payer ou vérifier. Lorsque les connaissances qui sous-tendent cette décision font défaut, un attaquant n'a pas à vaincre la technologie. Il peut simplement demander à une personne à qui l'on n'a jamais appris à dire non.

## Ce que le NIST appelle le continuum d'apprentissage

Des référentiels comme le NIST SP 800-50 décrivent l'apprentissage comme un continuum plutôt que comme un événement isolé. Chaque étape exige davantage que la précédente, et c'est en sautant l'étape intermédiaire que les programmes échouent le plus souvent.

| Étape | Public | À quoi ressemble la réussite |
| --- | --- | --- |
| Sensibilisation | Tout le monde | Remarquer que quelque chose cloche : un expéditeur usurpé, une invite de connexion inattendue |
| Formation | Personnes occupant un rôle précis | Une réponse exercée et correcte, par exemple la finance qui vérifie un changement de coordonnées bancaires via un numéro de téléphone connu |
| Éducation | Concepteurs et administrateurs | Un travail sûr par défaut, intégré à la façon dont le métier est exercé |

Un programme qui ne délivre que la première ligne laisse le personnel financier sans la moindre notion de la fraude à la facture, et les développeurs sans la moindre notion des secrets laissés dans le code. Ces déficits plus profonds alimentent directement les [erreurs de conception et de développement](design-and-development-errors.md) et la [mauvaise configuration](misconfiguration.md), où la cause racine est souvent une personne à qui l'on n'a jamais montré la valeur par défaut sûre.

## Suivre un seul e-mail de phishing à travers une équipe non formée

Le coût de ce déficit se voit le plus facilement en suivant un seul message :

1. **Il arrive.** Un e-mail imite un fournisseur connu et demande à un employé comptable de mettre à jour les coordonnées de paiement "avant la prochaine échéance".
2. **Il est cru.** Personne n'a jamais expliqué que l'urgence associée à un changement de paiement est le schéma de fraude classique, si bien que l'employé le traite comme une routine.
3. **Il donne lieu à une action.** Les coordonnées sont modifiées. Aucun filtre n'a signalé le message, car rien en lui n'était techniquement malveillant.
4. **Il est dissimulé.** Lorsque l'erreur apparaît quelques jours plus tard, l'employé craint d'être blâmé et garde le silence, de sorte que la perte s'aggrave avant que quiconque ne réagisse.

La sensibilisation aurait arrêté le message à l'étape deux. Une habitude de signalement sans blâme aurait raccourci l'étape quatre. L'une comme l'autre s'apprennent, elles ne s'installent pas.

## La courbe de l'oubli face au module annuel

Le programme le plus répandu est un unique cours annuel, et il lutte contre la mémoire humaine. Les personnes oublient la plupart de ce qu'elles entendent en quelques jours, à moins que cela ne soit rafraîchi et utilisé. La solution n'est pas un cours plus long mais un rythme plus régulier : de courtes piqûres de rappel, des exemples tirés des menaces que chaque équipe rencontre réellement, et des simulations de phishing ou des exercices sur table qui obligent à pratiquer la reconnaissance plutôt qu'à seulement la lire. Les règles inexpliquées s'érodent de la même façon ; une personne qui ne comprend pas pourquoi un contrôle existe tend à le contourner, une dérive examinée dans [comportement des utilisateurs et usages abusifs](user-behavior-and-misuse.md).

## Mesurer la reconnaissance, pas seulement les clics

Les taux de clic des simulations sont visibles mais superficiels : un faible taux peut seulement signifier que le test était facile. Le signal le plus fort est le signalement. Une personne qui repère un message suspect et le signale devient un capteur, donnant aux défenseurs un avertissement que les outils ont manqué. Suivre combien de personnes signalent, et à quelle vitesse, montre si la culture change réellement.

Aucun programme ne touche tout le monde, c'est pourquoi la formation s'associe à l'architecture. Les conceptions bâties sur le [moindre privilège](../principles/least-privilege.md) limitent ce qu'une seule décision mal informée peut exposer, de sorte qu'un système indulgent et une main-d'œuvre éduquée comblent mutuellement leurs lacunes.

## Références

- [NIST SP 800-50 Rev. 1, Building a Cybersecurity and Privacy Learning Program](https://csrc.nist.gov/pubs/sp/800/50/r1/final)
- [NIST SP 800-53 Rev. 5, Security and Privacy Controls (AT, Awareness and Training)](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [CISA: Recognize and Report Phishing](https://www.cisa.gov/secure-our-world/recognize-and-report-phishing)
- [UK NCSC: Phishing scams, how to spot and report them](https://www.ncsc.gov.uk/collection/phishing-scams)
- [ENISA Threat Landscape](https://www.enisa.europa.eu/topics/cyber-threats/threats-and-trends)
