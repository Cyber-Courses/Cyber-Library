---
title: "Responsabilité : relier chaque action à une identité responsable"
description: "Le principe de sécurité selon lequel chaque action significative peut être rattachée à une identité responsable grâce à une journalisation fiable et à la non-répudiation."
keywords:
  - responsabilité
  - journalisation d'audit
  - non-répudiation
  - traçabilité
  - signatures numériques
  - principes de sécurité
---

# Responsabilité

La **responsabilité** (accountability) est la propriété selon laquelle toute action significative dans un système peut être rattachée à l'identité qui en est responsable, avec des preuves suffisamment solides pour résister à l'examen. L'authentification établit qui est un acteur et l'autorisation décide ce qu'il peut faire ; la responsabilité enregistre ce qu'il a réellement fait, de sorte que le système puisse répondre plus tard à la question "qui a fait cela, et pouvons-nous le prouver ?" Sans elle, une violation, une erreur ou un litige devient un jeu de devinettes.

## Ce qu'apportent respectivement la journalisation, l'audit et la non-répudiation

Ces trois termes sont souvent employés de façon approximative, mais ils décrivent des niveaux de preuve différents, chacun s'appuyant sur le précédent.

| Mécanisme | Ce qu'il produit | Ce qu'il ne peut toujours pas faire |
|---|---|---|
| Journalisation | Un enregistrement horodaté indiquant qu'une identité a effectué une action | Prouver que l'enregistrement n'a pas été modifié par la suite |
| Audit | L'examen indépendant des enregistrements collectés pour établir ce qui s'est réellement passé | Empêcher l'acteur d'affirmer que le compte n'était pas le sien |
| Non-répudiation | Une preuve cryptographique liant un acteur précis à l'action | Aider si la clé de signature de l'acteur a été volée |

Une simple entrée de journal est une affirmation ; l'audit l'examine et la corrobore ; la non-répudiation la rend difficile à nier. Chaque couche coûte davantage à construire, c'est pourquoi la plupart des systèmes journalisent tout, auditent le sous-ensemble pertinent pour la sécurité, et réservent la non-répudiation aux actes à fort enjeu tels que les contrats, les paiements et les publications de code.

## Retracer un virement frauduleux

Supposons qu'une équipe financière découvre qu'un paiement non autorisé a quitté le compte de l'entreprise. La responsabilité transforme cette alerte en une piste que l'enquêteur peut suivre :

1. **Le système de paiement journalise le virement**, en enregistrant le montant, le bénéficiaire, l'heure et le compte `a.osei` qui l'a approuvé, plutôt qu'un identifiant partagé `finance`.
2. **Le journal du fournisseur d'identité** montre que `a.osei` s'est authentifié avec une clé matérielle quelques minutes plus tôt, depuis un pays inhabituel.
3. **Les horloges concordent**, car chaque système se synchronise sur la même source de temps de confiance, de sorte que la connexion et l'approbation s'enchaînent dans le bon ordre.
4. **L'approbation portait une signature numérique**, de sorte que l'acteur ne peut pas prétendre de façon crédible que quelqu'un a fabriqué la requête en son nom.

La piste pointe désormais vers un compte et un instant précis. Notez ce qu'elle ne prouve pas encore : les entrées identifient un compte, pas une personne, de sorte que des identifiants volés ou une session détournée auraient exactement la même apparence. Supprimez un seul maillon, comme un identifiant partagé ou des journaux stockés sur le serveur compromis lui-même, et la piste s'effondre en un haussement d'épaules.

## La tension avec la confidentialité et la minimisation des données

La responsabilité pousse à enregistrer davantage ; une bonne pratique de confidentialité pousse à enregistrer moins. La solution n'est pas de tout journaliser "au cas où". Cela gonfle le stockage, noie les signaux réels dans le bruit, et transforme les pistes d'audit en un jeu de données sensible à part entière, une cible tentante et une charge de conformité. La discipline de la [minimisation](minimization.md) s'applique ici aussi : décidez à l'avance quels événements sont pertinents pour la sécurité, capturez-les de façon cohérente, protégez-les, et fixez une durée de conservation qui serve l'enquête et la réglementation sans tout thésauriser. Un accès restreint selon le [moindre privilège](least-privilege.md) allège le compromis, car moins de personnes peuvent effectuer une action sensible, de sorte que la liste des acteurs plausibles est courte avant même d'ouvrir un seul journal.

## Où elle renforce les autres principes

La responsabilité repose sur l'[authentification et l'autorisation](authentication-and-authorization.md), puisqu'une piste n'est digne de confiance que dans la mesure où le sont les identités qu'elle nomme. Elle est la moins coûteuse lorsqu'elle est planifiée dès le départ, ce qui en fait une préoccupation de la [sécurité dès la conception](security-by-design.md) plutôt qu'un ajout tardif, et les architectures fondées sur le [zero trust](zero-trust.md) émettent naturellement les enregistrements par requête dont dépend l'attribution.

## Références

- [NIST Glossary: Accountability](https://csrc.nist.gov/glossary/term/accountability)
- [NIST Glossary: Non-repudiation](https://csrc.nist.gov/glossary/term/non_repudiation)
- [NIST SP 800-53 Rev. 5, Security and Privacy Controls (AU Audit and Accountability)](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [NIST SP 800-92, Guide to Computer Security Log Management](https://csrc.nist.gov/pubs/sp/800/92/final)
- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
- [ISO/IEC 27001 Information Security Management](https://www.iso.org/standard/27001)
