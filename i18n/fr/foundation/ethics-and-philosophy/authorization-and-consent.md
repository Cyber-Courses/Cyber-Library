---
title: "Autorisation et consentement : la frontière entre test et intrusion"
description: "La permission explicite est ce qui sépare un test de sécurité légitime d'une intrusion illégale, en définissant la portée, les limites et la légitimité de chaque action qu'entreprend un praticien."
keywords:
  - autorisation
  - consentement
  - portée de l'engagement
  - règles d'engagement
  - test responsable
  - éthique de la sécurité
---

# Autorisation et consentement

**L'autorisation est la permission explicite qui transforme un acte autrement hostile en un acte légitime.** La même technique, comme scanner un réseau, casser un mot de passe ou exploiter une faille, relève soit du travail professionnel de sécurité, soit du délit, et la différence réside rarement dans la technique elle-même. Elle réside dans le fait que le propriétaire du système a accepté, au préalable et par écrit, de laisser quelqu'un agir contre lui. Le consentement est le fondement éthique et juridique sur lequel repose chaque praticien.

## Même scan, deux actes différents

Imaginez deux personnes qui lancent exactement le même scan de ports et la même tentative de password spraying contre la même entreprise, la même nuit. Rien dans le trafic réseau ne permet de les distinguer. Ce qui diffère, c'est tout ce qui entoure les frappes au clavier.

| | Testeur autorisé | Personne sans autorisation |
|---|---|---|
| Statut juridique | Travail licite encadré par un accord signé | Infraction probable au regard des lois sur l'usage abusif de l'informatique |
| Responsabilité | Protégé par le contrat, dans les limites de ses termes | Personnellement exposé pour chaque action menée |
| Traçabilité | Le propriétaire sait ce qui a été convenu, par qui et quand | Le propriétaire l'apprend par des alertes, s'il l'apprend un jour |
| Autonomie du propriétaire | Respectée : le propriétaire a choisi d'être testé | Bafouée : quelqu'un a décidé à sa place |

Cette dernière ligne est le cœur éthique du sujet. Les personnes et les organisations ont le droit de décider qui peut sonder leurs systèmes et leurs données, et agir sans cette décision leur retire ce choix, aussi bonnes que soient les intentions. C'est aussi pourquoi la [connaissance à double usage](dual-use-knowledge.md) est encadrée par la permission plutôt que par la compétence : le défenseur et l'attaquant disposent des mêmes capacités.

## Qui a la légitimité pour dire oui

Une signature ne vaut que par l'autorité qui la soutient. Un consentement donné par la mauvaise personne peut laisser un testeur tout aussi exposé qu'une absence totale de consentement.

- **Le responsable sans légitimité.** Un chef d'équipe peut approuver le test d'une application qui tourne en réalité sur des systèmes partagés, réglementés ou à l'échelle du groupe, qu'il ne contrôle pas. Son accord ne s'étend pas à ces systèmes.
- **Le propriétaire derrière le locataire.** Une application testée tourne souvent sur une infrastructure cloud ou fournisseur dont le propriétaire n'a jamais rien signé. Les fournisseurs fixent leurs propres conditions : AWS, par exemple, permet à ses clients de tester leurs propres ressources sur une liste de services autorisés sans approbation préalable, mais interdit de tester l'infrastructure d'AWS elle-même et proscrit des activités comme l'inondation par déni de service sans approbation distincte.
- **Les personnes derrière les données.** Lorsqu'un test touche à de vrais dossiers de clients ou d'employés, le consentement du propriétaire du système ne couvre pas automatiquement les individus dont les données sont exposées. C'est là que l'autorisation rejoint la question de la [sécurité versus la vie privée](security-versus-privacy.md).

Savoir qui est en droit d'accorder l'accès à un système est en soi une question de pouvoir, approfondie dans [liberté versus contrôle](freedom-versus-control.md).

## Ce que fixe un document de règles d'engagement

L'autorisation est consignée avant le début de tout travail, généralement dans un document de périmètre ou un accord de règles d'engagement. Un bon document répond par écrit à quatre questions :

1. **Qui** a accordé la permission, et avait-il réellement l'autorité pour l'accorder ?
2. **Quels** systèmes, adresses et comptes sont dans le périmètre, et lesquels sont explicitement exclus ?
3. **Quand** le test peut-il avoir lieu, et dans quelle fenêtre de temps ?
4. **Jusqu'où** un testeur peut-il aller : une simple preuve d'accès, ou une exploitation active et une récupération de données ?

Des référentiels de test comme le NIST SP 800-115 considèrent cette phase de planification comme une partie intégrante de l'évaluation, et non comme une formalité administrative préalable. Une permission qui n'est pas documentée est, en pratique, une permission qui n'existe pas : "tout le monde sait qu'on teste ça" n'est pas un consentement.

## Bug bounties et safe harbor : un consentement publié pour tous

Un test d'intrusion accorde le consentement à une équipe nommément désignée. Les programmes de bug bounty et les politiques de divulgation coordonnée des vulnérabilités fonctionnent autrement : ils publient un consentement valable pour quiconque respecte les règles. Une politique de safe harbor (refuge juridique) indique aux chercheurs ce qu'ils peuvent tester, quelles techniques sont proscrites, comment signaler leurs découvertes, et ce que l'organisation promet en retour, généralement de ne pas engager de poursuites contre une recherche menée de bonne foi dans le respect de ces conditions.

Ces conditions restent contraignantes. Un chercheur qui exfiltre plus de données que nécessaire pour démontrer une faille, ou qui teste un actif non mentionné dans la politique, sort du consentement publié tout aussi sûrement qu'un testeur sous contrat qui ignore son périmètre.

## Trois moments où la frontière se déplace

La frontière nette s'estompe dans le travail réel, et c'est là que le jugement compte le plus.

**Le pivot.** Un testeur exploite un serveur web inclus dans le périmètre et y trouve des identifiants qui ouvrent l'accès à un serveur de base de données jamais mentionné dans l'accord. La découverte peut avoir de la valeur, mais la permission s'arrête à la frontière. La bonne démarche consiste à consigner le chemin, à s'arrêter et à demander au client s'il souhaite étendre le périmètre.

**Le voisin inattendu.** Une plage d'adresses IP incluse dans le périmètre contient en réalité des adresses désormais louées à une autre entreprise. Le périmètre sur le papier ne correspond pas à la réalité, et ces hôtes appartiennent à quelqu'un qui n'a jamais consenti.

**Le oui verbal.** Un interlocuteur côté client lance au cours d'un appel : "Allez-y, essayez aussi le VPN." Tant que cela n'est pas confirmé par écrit par une personne habilitée, il s'agit d'un consentement implicite, pas d'une autorisation.

Dans chaque cas, le réflexe éthique par défaut est le même : lorsque l'autorisation n'est pas claire, s'arrêter et demander, plutôt que poursuivre et s'expliquer plus tard. Même un travail pleinement autorisé laisse le praticien comptable de ses conséquences, ce qui fait l'objet de la [responsabilité morale](moral-responsibility.md).

## Références

- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [NIST SP 800-115, Technical Guide to Information Security Testing and Assessment](https://csrc.nist.gov/pubs/sp/800/115/final)
- [CISA: Coordinated Vulnerability Disclosure Process](https://www.cisa.gov/coordinated-vulnerability-disclosure-process)
- [EFF: Coders' Rights Project Vulnerability Reporting FAQ](https://www.eff.org/issues/coders/vulnerability-reporting-faq)
- [AWS : politique relative aux tests d'intrusion](https://aws.amazon.com/security/penetration-testing/)
