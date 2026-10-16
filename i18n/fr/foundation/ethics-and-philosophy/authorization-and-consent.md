---
title: "Autorisation et consentement : la permission comme frontière entre test et intrusion"
description: "Comment la permission explicite sépare le test de sécurité légitime de l'intrusion, qui peut l'accorder, ce qu'elle doit couvrir, et quand elle prend fin."
keywords:
  - autorisation
  - consentement
  - règles d'engagement
  - périmètre de test
  - safe harbor
  - éthique de la sécurité
---

# Autorisation et consentement

**L'autorisation est la permission explicite qui transforme un acte autrement hostile en travail légitime.** Un balayage de ports, un mot de passe cassé ou une faille exploitée peut être une mission rémunérée ou une infraction, et rien dans les paquets ne permet de les distinguer. La différence se situe entièrement en dehors du clavier : le propriétaire du système a-t-il accepté, à l'avance, d'être visé ? Le consentement est le terrain sur lequel se tient un praticien, et sans lui, même une technique irréprochable relève de l'intrusion.

## La permission, pas la technique, est la ligne de partage

Imaginez deux personnes exécutant le même balayage contre la même entreprise la même nuit. Le trafic est indiscernable ; la légitimité qui le sous-tend ne l'est pas.

| | Agir avec autorisation | Agir sans elle |
|---|---|---|
| Fondement juridique | Travail licite dans le cadre d'un accord signé | Une infraction probable au regard du droit sur la fraude informatique |
| Le choix du propriétaire | Respecté ; il a choisi d'être testé | Bafoué ; quelqu'un a décidé à sa place |
| Visibilité | Le propriétaire sait qui, quoi et quand | Le propriétaire l'apprend par les alertes, si tant est qu'il l'apprenne |

La ligne du milieu en est le cœur éthique. Les personnes et les organisations ont le droit de décider qui peut sonder leurs systèmes, et agir sans cette décision retire ce choix, quelle que soit la qualité de l'intention. C'est aussi pourquoi [le savoir à double usage](dual-use-knowledge.md) est régi par la permission et non par la compétence : le défenseur et l'intrus maîtrisent les mêmes techniques.

## Quel "oui" compte réellement

Une signature ne vaut que par l'autorité qui la sous-tend, et le consentement de la mauvaise partie laisse un testeur aussi exposé que l'absence totale de consentement.

- **Un responsable sans portée.** Un chef d'équipe peut approuver le test d'une application qui, en réalité, fonctionne sur des systèmes partagés ou réglementés qu'il ne contrôle pas. Son accord ne s'étend pas à ces systèmes.
- **Le propriétaire de l'infrastructure.** Les applications reposent souvent sur des plateformes cloud ou de fournisseurs dont l'opérateur n'a rien signé. Les fournisseurs cloud publient leurs propres règles de test, autorisant les clients à tester leurs propres ressources sur des services nommés tout en interdisant les attaques contre l'infrastructure du fournisseur lui-même.
- **Les personnes à l'intérieur des données.** Un test qui touche de vrais enregistrements clients atteint des individus pour qui le propriétaire du système ne peut pas consentir, ce qui est précisément là où l'autorisation rencontre [sécurité contre vie privée](security-versus-privacy.md).

Savoir qui est en droit d'accorder un accès est en soi une question de pouvoir, abordée dans [liberté contre contrôle](freedom-versus-control.md).

## Ce que les règles d'engagement doivent fixer

La permission est consignée avant le début de tout travail, généralement sous la forme d'un document de périmètre ou d'un accord de règles d'engagement. Un bon document répond par écrit à quatre questions :

1. **Qui** a accordé la permission, et avait-il l'autorité de l'accorder ?
2. **Quels** systèmes et comptes sont dans le périmètre, et lesquels sont explicitement interdits ?
3. **Quand** le test peut-il se dérouler, et à l'intérieur de quelle fenêtre ?
4. **Jusqu'où** peut-il aller : simple preuve d'accès, ou exploitation active et récupération de données ?

Des recommandations comme le NIST SP 800-115 considèrent cette planification comme faisant partie de l'évaluation, et non comme une formalité qui la précède. Une permission non documentée n'est, en pratique, pas une permission : "tout le monde sait qu'on teste ceci" n'est pas un consentement.

Les programmes de bug bounty et de divulgation coordonnée étendent la même idée à des inconnus. Une politique de safe harbor publie un consentement à l'intention de quiconque en respecte les termes, promettant de ne pas poursuivre la recherche de bonne foi qui reste dans les limites, mais un chercheur qui exfiltre plus que nécessaire ou sonde un actif non répertorié en est sorti tout aussi clairement qu'un prestataire ignorant le périmètre.

## Quand le "oui" prend fin en cours de mission

La ligne nette se brouille pendant le travail réel, et le discernement importe le plus aux limites.

1. **Le pivot.** Un serveur dans le périmètre livre des identifiants vers un hôte de base de données que personne n'a nommé. La trouvaille est tentante, mais la permission s'arrête à la frontière ; notez le chemin, arrêtez-vous et demandez avant d'aller plus loin.
2. **Le voisin surprise.** Une plage d'adresses dans le périmètre se révèle inclure des hôtes désormais loués à une autre entreprise qui n'a jamais consenti.
3. **Le oui verbal.** Un contact dit au téléphone : "essayez aussi le VPN". Un accord oral donné par quelqu'un qui a l'autorité de l'accorder est une permission réelle, mais tant qu'il n'est pas inscrit dans le périmètre, il ne peut être prouvé par la suite et peut ne pas émaner de quelqu'un pouvant réellement l'accorder, de sorte que le VPN reste hors des règles d'engagement jusqu'à ce qu'elles soient formellement amendées.

Par défaut, dans chaque cas, la règle est la même : lorsque l'autorisation n'est pas claire, arrêtez-vous et demandez plutôt que de poursuivre et de vous justifier plus tard. Même un travail pleinement autorisé laisse le praticien responsable de ses résultats, ce qui fait l'objet de [la responsabilité morale](moral-responsibility.md).

## Références

- [NIST SP 800-115, Technical Guide to Information Security Testing and Assessment](https://csrc.nist.gov/pubs/sp/800/115/final)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [CISA: Coordinated Vulnerability Disclosure Process](https://www.cisa.gov/coordinated-vulnerability-disclosure-process)
- [EFF: Coders' Rights Project Vulnerability Reporting FAQ](https://www.eff.org/issues/coders/vulnerability-reporting-faq)
