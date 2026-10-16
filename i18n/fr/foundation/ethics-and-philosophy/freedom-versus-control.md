---
title: "Liberté contre contrôle : l'arbitrage central de la sécurité"
description: "Comment le travail de sécurité met en balance l'accès ouvert et l'autonomie individuelle face aux restrictions et à la surveillance qu'impose la protection."
keywords:
  - liberté contre contrôle
  - liberté négative et positive
  - arbitrages de sécurité
  - restrictions d'accès
  - autonomie et surveillance
  - cliquet du contrôle
---

# Liberté contre contrôle

**La liberté et le contrôle sont deux biens, et le travail de sécurité dépense l'un pour acheter l'autre.** Les gens veulent des systèmes ouverts, utilisables et à l'abri du regard de l'exploitant ; ces mêmes systèmes doivent résister aux usages abusifs, ce qui implique de restreindre les actions, de surveiller l'activité et de retirer des décisions aux mains des individus. Presque chaque choix de sécurité se situe quelque part sur la ligne qui sépare ces deux biens, et aucun point de cette ligne n'est exempt de coût.

## La liberté négative et positive, appliquée à un réseau

Le philosophe Isaiah Berlin, dans sa conférence de 1958 "Deux conceptions de la liberté", a divisé la liberté en deux types. La *liberté négative* est la liberté à l'égard de toute ingérence : personne ne vous empêche d'agir. La *liberté positive* est la liberté de réellement accomplir quelque chose : disposer de la capacité et des conditions pour bien agir.

La sécurité touche aux deux, et les deux peuvent pointer dans des directions opposées :

- Un gestionnaire de mots de passe obligatoire, des mises à jour forcées et des ports USB bloqués **réduisent la liberté négative**. L'utilisateur subit une ingérence.
- Les mêmes mesures peuvent **élargir la liberté positive**. Un réseau qui n'est pas constamment compromis est un réseau que les gens peuvent effectivement utiliser pour faire leur travail.

Les cas difficiles sont ceux où protéger la liberté positive d'une personne exige de porter atteinte à la liberté négative d'une autre. "Plus de sécurité" est rarement un gain net ; c'est généralement un transfert.

## Le cadran, pas l'interrupteur

Traiter la liberté et le contrôle comme un tout ou rien masque le vrai travail, qui consiste à choisir un réglage. La même ressource peut se situer en des points très différents selon qui la détient et pourquoi.

| Réglage | Liberté privilégiée | Contrôle privilégié |
|---|---|---|
| Accès par défaut | Ouvert sauf restriction | Refusé sauf autorisation |
| Surveillance | Journaux réduits au minimum | Activité enregistrée en détail |
| Exceptions | L'utilisateur peut s'auto-approuver | Chaque demande est examinée |
| Partage du savoir | Publié ouvertement | Partagé selon le besoin d'en connaître |

La plupart des techniques de sécurité sont un déplacement délibéré sur l'une de ces lignes, et non un saut jusqu'à l'extrémité de toutes.

## Faire monter le cadran d'un réseau de campus

Prenons l'exemple d'une université qui ajoute des contrôles à un réseau de recherche après un incident.

1. **Départ.** N'importe quel membre peut connecter n'importe quel appareil et atteindre n'importe quel système. Liberté maximale, assurance minimale.
2. **Premier tour.** Les appareils doivent être enregistrés et corrigés avant de rejoindre le réseau. Un gain clair en matière de sécurité ; une petite taxe sur l'autonomie.
3. **Deuxième tour.** L'accès est accordé par rôle et expire, appliquant le [moindre privilège](../principles/least-privilege.md). Les chercheurs déposent désormais des demandes qu'ils avaient l'habitude de sauter.
4. **Dépassement.** La journalisation complète des frappes au clavier est ajoutée "par précaution". La protection augmente légèrement ; la confiance et le moral chutent fortement.

L'étape quatre montre pourquoi la destination importe : au-delà d'un certain point, chaque nouvelle restriction achète peu de sécurité et dépense beaucoup de bonne volonté.

## Le cliquet, et la tension sous d'autres noms

Les contrôles ont une qualité de cliquet : ils sont faciles à ajouter après un incident et politiquement difficiles à retirer, de sorte que les systèmes dérivent vers le contrôle à moins que quelqu'un ne plaide pour le camp de la liberté. Nommer à voix haute les valeurs concurrentes est ce qui maintient le cadran honnête.

Cette même tension réapparaît dans toute la fondation sous d'autres étiquettes. C'est la [sécurité contre la vie privée](security-versus-privacy.md) lorsque le contrôle en question est la surveillance, et c'est l'[autorisation et le consentement](authorization-and-consent.md) lorsque la question est de savoir qui peut tracer la ligne et si les gouvernés y ont consenti. Elle façonne aussi le [savoir à double usage](dual-use-knowledge.md), où restreindre l'information pour prévenir un préjudice restreint aussi les défenseurs qui en ont besoin.

## Références

- [Isaiah Berlin, *Liberty* (comprenant "Two Concepts of Liberty"), Oxford University Press](https://global.oup.com/academic/product/liberty-9780199249893)
- [NIST SP 800-53 Rev. 5, Security and Privacy Controls (famille Access Control)](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [Saltzer and Schroeder, The Protection of Information in Computer Systems (1975)](https://www.cs.virginia.edu/~evans/cs551/saltzer/)
- [NIST SP 800-37 Rev. 2, Risk Management Framework](https://csrc.nist.gov/pubs/sp/800/37/r2/final)
- [EFF, Surveillance Self-Defense](https://ssd.eff.org/)
