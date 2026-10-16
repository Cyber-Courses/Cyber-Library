---
title: "Liberté contre contrôle : la tension fondamentale du travail de sécurité"
description: "Comment les praticiens de la sécurité mettent en balance l'accès ouvert, l'autonomie et la libre circulation de l'information avec les restrictions et la supervision qu'exige la protection."
keywords:
  - liberté contre contrôle
  - accès ouvert
  - compromis de sécurité
  - autonomie et supervision
  - liberté de l'information
  - éthique de la sécurité
---

# Liberté contre contrôle

**Chaque décision de sécurité trace une ligne** entre ce que les personnes sont libres de faire et ce que le système les empêche de faire. L'accès ouvert, l'autonomie et la libre circulation de l'information rendent les systèmes utiles, accueillants et innovants. Le contrôle, sous la forme de restrictions, de surveillance et de filtrage, les rend plus sûrs. Les deux tirent en sens opposés, et aucun réglage ne satisfait les deux à la fois. Apprendre à reconnaître cette tension, plutôt que de prétendre qu'un camp l'emporte toujours, est une compétence fondamentale pour quiconque travaille dans la sécurité.

## Le prix de chaque garde-fou

La sécurité existe pour protéger les personnes et les données, mais la protection est rarement gratuite. Chaque garde-fou consomme quelque chose auquel l'utilisateur tient, et il est utile de nommer ce coût explicitement :

| Contrôle | Ce qu'il protège | Ce qu'il coûte |
| --- | --- | --- |
| Une authentification | Le compte et ses données | De la friction à chaque visite |
| Un filtre de contenu | Les utilisateurs et les machines, contre les logiciels malveillants | Du travail légitime bloqué par erreur |
| La journalisation et la surveillance | La capacité à repérer les intrus | La vie privée des utilisateurs ordinaires, dont les actions sont aussi enregistrées |
| Un circuit d'approbation | Les systèmes, contre les changements risqués | La rapidité, puisque les changements honnêtes attendent aussi longtemps que les dangereux |

Internet lui-même a été conçu pour faire circuler l'information librement, et une grande partie de sa valeur vient encore de cette ouverture. Le contrôle est souvent ce que l'on ajoute ensuite, et quelqu'un le paie toujours en commodité, en vie privée ou en autonomie.

## Trois endroits où la ligne ne cesse de bouger

**Facilité d'usage contre verrouillage.** Les équipes ajustent sans cesse le degré de rigueur d'un système. Des règles trop souples invitent aux abus ; des règles trop strictes poussent les utilisateurs vers des contournements moins sûrs que le comportement qu'ils remplacent. Le National Cyber Security Centre britannique le souligne à propos des politiques de mots de passe : des règles trop complexes conduisent les gens à réutiliser leurs mots de passe, à les noter de façon peu sûre ou à en choisir de prévisibles. Saltzer et Schroeder ont nommé l'idée sous-jacente dès 1975, sous le terme d'"acceptabilité psychologique" : une protection ne fonctionne que si les personnes peuvent l'appliquer couramment et correctement.

**Transparence contre secret.** Publier le fonctionnement d'une défense invite à un examen qui la renforce, mais peut aussi fournir une carte aux attaquants. Le dissimuler protège la conception tout en affaiblissant la confiance et l'examen du public. Le principe de "conception ouverte" de Saltzer et Schroeder penche du côté de la transparence : un mécanisme ne doit pas dépendre de l'ignorance des attaquants, mais seulement de clés ou de mots de passe plus faciles à protéger. La même ouverture qui diffuse le savoir défensif diffuse aussi les moyens d'attaquer, ce qui est le sujet du [savoir à double usage](dual-use-knowledge.md).

**Autonomie individuelle contre sécurité collective.** La liberté d'une seule personne d'installer n'importe quel logiciel ou de partager n'importe quel fichier peut exposer tous ceux qui partagent le réseau. Restreindre cette liberté protège le groupe, mais traite chacun de ses membres comme un risque potentiel.

## Un cas concret : verrouiller les ordinateurs portables du bureau

Prenons une petite entreprise qui décide, après une alerte aux logiciels malveillants, d'empêcher ses employés d'installer eux-mêmes des logiciels.

1. **Le contrôle.** Seule l'équipe informatique peut installer des applications. Le risque qu'un utilisateur télécharge un outil malveillant chute fortement.
2. **Le premier coût.** Un graphiste a besoin d'un utilitaire de polices pour une échéance client et attend deux jours l'approbation. Le travail ralentit et la frustration monte.
3. **Le contournement.** Certains employés commencent à travailler sur leurs ordinateurs personnels ou dans des outils web non vérifiés que l'équipe informatique ne voit pas du tout. Les données que la règle devait protéger se trouvent désormais dans un endroit moins sûr.
4. **L'ajustement.** L'équipe informatique publie une liste d'outils préapprouvés que chacun peut installer, conserve l'examen manuel pour tout le reste et s'engage à répondre dans la journée. La liberté est en partie rétablie, et le contrôle couvre toujours les cas les plus risqués.

Ni la politique ouverte d'origine ni le verrouillage strict n'étaient la bonne réponse. Le réglage viable est venu de l'observation de la manière dont les gens réagissaient réellement, et d'ajustements faits avec eux.

## Qui trace la ligne

Les zones véritablement contestées sont celles où des personnes raisonnables ne sont pas d'accord, et elles portent surtout sur l'autorité plutôt que sur la technologie :

- **Qui décide.** Un contrôle qui semble protecteur pour un administrateur peut ressembler à de la surveillance pour la personne observée. La légitimité dépend du consentement et de qui détient l'autorité pour fixer les limites, la question explorée dans [autorisation et consentement](authorization-and-consent.md). Lorsque c'est la surveillance elle-même qui pose problème, la tension se resserre en [sécurité contre vie privée](security-versus-privacy.md).
- **Les réglages par défaut portent des valeurs.** La plupart des gens ne modifient jamais les réglages par défaut, de sorte que livrer un système ouvert ou verrouillé décide discrètement de l'équilibre pour des millions de personnes.
- **Le contrôle peut survivre à sa raison d'être.** Les restrictions ajoutées face à une menace réelle ont tendance à rester bien après la disparition de la menace, s'accumulant jusqu'à former des systèmes sûrs mais hostiles aux personnes qui s'y trouvent.

Le but n'est pas de maximiser le contrôle ou la liberté, mais de faire ce compromis de manière délibérée, visible et avec les personnes qu'il concerne. Décider où tracer la ligne est un acte éthique dont le praticien répond, ce qui le relie directement à la [responsabilité morale](moral-responsibility.md).

## Références

- [NIST SP 800-53 Rev. 5, Security and Privacy Controls for Information Systems and Organizations](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [Saltzer and Schroeder, The Protection of Information in Computer Systems (1975)](https://www.cs.virginia.edu/~evans/cs551/saltzer/)
- [UK National Cyber Security Centre : Password policy, updating your approach](https://www.ncsc.gov.uk/collection/passwords/updating-your-approach)
- [Electronic Frontier Foundation : Surveillance Self-Defense](https://ssd.eff.org/)
- [Electronic Frontier Foundation : Privacy](https://www.eff.org/issues/privacy)
