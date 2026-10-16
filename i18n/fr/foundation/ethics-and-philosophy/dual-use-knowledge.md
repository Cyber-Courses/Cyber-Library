---
title: "Connaissance à double usage : la même compétence défend et attaque"
description: "En sécurité, une information ou une technique peut défendre ou nuire selon qui la détient et dans quel but elle est utilisée."
keywords:
  - connaissance à double usage
  - technologie à double usage
  - sécurité offensive
  - divulgation responsable
  - éthique de la recherche en sécurité
  - connaissance et préjudice
---

# Connaissance à double usage

**La connaissance à double usage** est une information ou une technique qui peut défendre ou nuire selon qui la détient et pourquoi. La même compréhension qui permet à un défenseur de trouver et de corriger une faiblesse permet à un attaquant de la trouver et de l'exploiter. La connaissance elle-même est neutre. Son effet provient de l'intention, du contexte et des conséquences, ce qui explique pourquoi les questions de [responsabilité morale](moral-responsibility.md) et d'[autorisation et consentement](authorization-and-consent.md) sont si proches de ce sujet.

## Une compétence, deux directions

Presque chaque compétence fondamentale en sécurité possède son reflet. La technique ne change pas d'une colonne à l'autre ; seuls changent celui qui la détient, la cible et la permission.

| Compétence | Usage défensif | Usage offensif |
| --- | --- | --- |
| Cartographie réseau et scan de ports | Un administrateur inventorie les services exposés | Un intrus cherche un point d'entrée |
| Développement d'exploits | Un testeur prouve qu'une faille est réelle pour qu'elle soit corrigée, et les défenseurs écrivent des détections pour elle | Un attaquant transforme la faille en accès |
| Rétro-ingénierie de logiciels malveillants | Un analyste extrait des indicateurs et construit des signatures | Un criminel étudie le code d'un rival pour en réutiliser les astuces |
| Cassage de mots de passe | Un auditeur mesure la faiblesse des mots de passe d'une organisation | Un voleur récupère des identifiants dans une base de données dérobée |
| Techniques d'hameçonnage | Une équipe de sécurité mène des simulations de sensibilisation | Un fraudeur vole des identifiants de connexion |

## Pourquoi la défense ne peut éviter d'apprendre l'attaque

On ne peut pas se défendre contre une technique que l'on ne comprend pas. Les modèles de menaces, le red teaming, les règles de détection et les correctifs exigent tous de savoir comment un système se brise, de sorte que le domaine dépend de personnes qui construisent, partagent et publient des connaissances offensives.

L'ouverture est aussi ce qui fait progresser le domaine. La recherche publique, l'outillage partagé et les failles divulguées élèvent le niveau de référence de tous, et un défenseur travaillant seul aurait toujours un temps de retard sur des attaquants qui partagent librement entre eux. Le prix à payer est que l'on ne choisit pas son public : une conférence, un billet de blog ou un outil publié atteint aussi bien les défenseurs que les attaquants.

## Leviers pour diffuser ce que l'on sait

Les praticiens gèrent la connaissance à double usage par des normes et du discernement plutôt que par une règle unique. Les principaux leviers, en allant à peu près du choix individuel à la politique d'État :

1. **Divulgation coordonnée.** Un chercheur signale une faille au fournisseur et lui laisse le temps de la corriger avant d'en publier les détails. Certaines équipes fixent des délais stricts ; Google Project Zero, par exemple, accorde 90 jours aux fournisseurs pour publier un correctif, puis rend la faille publique.
2. **Retenue sur la preuve de concept.** Démontrer qu'une faiblesse existe sans livrer une arme clé en main pour l'exploiter.
3. **Outillage avec garde-fous.** Les cadriciels offensifs existent ouvertement, mais leurs auteurs documentent l'usage légitime et les limites légales.
4. **Contrôles à l'exportation.** Certaines capacités sont traitées comme des biens contrôlés. En 2013, l'Arrangement de Wassenaar, un régime multilatéral de contrôle des exportations d'armes conventionnelles et de biens et technologies à double usage, a ajouté certains logiciels d'intrusion à ses listes de contrôle.

## Cas pratique : une faille dans un routeur que personne ne corrige

Un chercheur découvre une faille permettant la prise de contrôle à distance d'un routeur domestique dont le fabricant a cessé le support. Chaque option a un coût :

- **Signaler et attendre.** Le fournisseur risque de ne jamais répondre ; attendre ne protège alors personne et laisse les propriétaires dans l'ignorance.
- **Publier tous les détails.** Les propriétaires et les fournisseurs d'accès apprennent qu'ils doivent remplacer ou isoler l'appareil, mais les attaquants obtiennent une recette au même moment.
- **Publier un avertissement sans détail technique.** Cela réduit le risque d'imitateurs, mais complique la tâche des défenseurs qui doivent écrire des détections ou confirmer leur exposition.
- **Vendre la découverte.** Vendre une capacité est licite dans bien des endroits, mais l'acheteur peut être une équipe de sécurité, un gouvernement ou un courtier, et le vendeur maîtrise rarement où elle finit.

Ce cas montre où des personnes raisonnables ne sont pas d'accord. Le détail aide les défenseurs à agir vite et les attaquants à agir plus vite encore. Les délais de divulgation supposent que les fournisseurs peuvent et veulent corriger, ce qui n'est pas toujours vrai pour les infrastructures critiques ou les logiciels abandonnés. Et restreindre entièrement la connaissance peut dissuader la recherche légitime sans arrêter les adversaires déterminés qui ignorent les règles, un thème approfondi dans [liberté contre contrôle](freedom-versus-control.md). La même ambivalence apparaît dans [sécurité contre vie privée](security-versus-privacy.md), où une même capacité de surveillance sert à la fois la protection et l'intrusion.

Il n'existe aucun réglage qui rende la connaissance inoffensive. La responsabilité tient à la manière dont elle est partagée et utilisée, non au fait de prétendre qu'elle pourrait rester inconnue.

## Références

- [CISA: Coordinated Vulnerability Disclosure Process](https://www.cisa.gov/coordinated-vulnerability-disclosure-process)
- [FIRST Guidelines and Practices for Multi-Party Vulnerability Coordination and Disclosure](https://www.first.org/global/sigs/vulnerability-coordination/multiparty/guidelines-v1.1)
- [Google Project Zero: Vulnerability Disclosure Policy](https://googleprojectzero.blogspot.com/p/vulnerability-disclosure-policy.html)
- [The Wassenaar Arrangement: Control Lists](https://www.wassenaar.org/control-lists/)
- [Wikipédia : Technologie à double usage](https://en.wikipedia.org/wiki/Dual-use_technology)
