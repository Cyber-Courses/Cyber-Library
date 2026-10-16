---
title: "Triade CIA : confidentialité, intégrité et disponibilité"
description: "Les trois propriétés de sécurité, confidentialité, intégrité et disponibilité, qui définissent ensemble ce que signifie, pour un système et ses données, être sécurisés."
keywords:
  - triade cia
  - confidentialité
  - intégrité
  - disponibilité
  - propriétés de sécurité
  - sécurité de l'information
---

# Triade CIA

**La triade CIA** désigne les trois propriétés que la sécurité vise à protéger : la confidentialité, l'intégrité et la disponibilité. C'est le modèle le plus ancien et le plus répandu pour réfléchir à ce que "sécurisé" veut réellement dire. Plutôt que de traiter la sécurité comme une qualité unique, la triade la décompose en trois objectifs distincts, de sorte qu'un contrôle, un risque ou un incident puisse être décrit selon la propriété qu'il touche. Quand on dit qu'un système a été compromis, on veut presque toujours dire qu'une ou plusieurs de ces trois propriétés ont été mises en défaut.

## Trois questions à poser pour chaque actif

Chaque propriété répond à une question différente sur une information ou un système. Les [actifs](assets-and-attack-surface.md) que vous protégez méritent de l'être précisément parce qu'un "non" à l'une de ces questions ferait des dégâts.

| Propriété | La question posée | À quoi ressemble une défaillance | Contrôles qui la servent |
| --- | --- | --- | --- |
| Confidentialité | Seules les bonnes personnes peuvent-elles la lire ? | Une fuite de données expose les dossiers des clients | Chiffrement, contrôles d'accès, authentification |
| Intégrité | Est-elle exacte, et saurions-nous si elle avait été modifiée ? | Un solde bancaire est falsifié, un fichier de configuration est modifié en silence | Sommes de contrôle, signatures numériques, historique des versions |
| Disponibilité | Les utilisateurs autorisés peuvent-ils y accéder quand ils en ont besoin ? | Un rançongiciel ou une attaque par déni de service met un service hors ligne | Redondance, sauvegardes, planification des capacités |

La loi fédérale américaine, telle que citée dans FIPS 199, en donne les définitions formelles. La confidentialité consiste à préserver les restrictions autorisées sur l'accès à l'information et sur sa divulgation. L'intégrité consiste à se prémunir contre la modification ou la destruction indue de l'information, et inclut explicitement la non-répudiation et l'authenticité. La disponibilité consiste à garantir un accès et une utilisation de l'information fiables et en temps voulu.

## Comment chaque propriété est mise en défaut

Ces défaillances méritent d'être étudiées séparément, car elles proviennent de [menaces](threats-and-threat-actors.md) différentes et appellent des [contrôles](controls-and-countermeasures.md) différents.

- **La confidentialité est rompue par la divulgation.** Des identifiants volés, un espace de stockage mal configuré, un initié qui copie des fichiers, ou du trafic envoyé en clair sur un réseau. Les données sont toujours là et toujours exactes, mais quelqu'un qui ne devrait pas les avoir les détient désormais, et c'est irréversible.
- **L'intégrité est rompue par une modification non autorisée ou non détectée.** Un attaquant qui modifie des enregistrements, un logiciel malveillant qui altère un binaire, ou un bogue qui corrompt une base de données. Le danger est souvent que personne ne s'en aperçoit : un lecteur doit pouvoir être sûr que les données sont bien ce qu'elles prétendent être, et une falsification silencieuse détruit cette confiance.
- **La disponibilité est rompue par le déni d'accès.** Un afflux massif de trafic, des fichiers chiffrés contre rançon, une sauvegarde supprimée, ou tout simplement une coupure de courant. La sécurité ne consiste pas seulement à tout verrouiller : des données que personne ne peut atteindre n'ont aucune valeur pour ceux qui en dépendent.

Une [vulnérabilité](vulnerabilities-and-exploits.md) est généralement évaluée selon la ou les propriétés qu'elle permet à un attaquant de compromettre, et selon la gravité de l'atteinte.

## Un incident de rançongiciel, lu de trois façons

Prenons une clinique frappée par un rançongiciel moderne à "double extorsion". Les attaquants copient d'abord les dossiers des patients sur leurs propres serveurs, puis chiffrent tout ce qui se trouve sur le réseau de la clinique.

1. **La disponibilité tombe de la façon la plus visible.** Le personnel ne peut plus ouvrir les plannings ni les dossiers des patients, et les rendez-vous sont annulés.
2. **La confidentialité a déjà été compromise.** Les dossiers exfiltrés sont désormais entre les mains de criminels, et la menace de les publier constitue le second levier de l'extorsion. La restauration depuis une sauvegarde n'y change rien.
3. **L'intégrité est désormais mise en doute.** Même après la reprise, la clinique doit se demander si des dossiers ont été modifiés pendant que les attaquants avaient accès au réseau. Sans journaux fiables ni sauvegardes signées, elle pourrait être incapable de répondre.

Décrire l'incident de cette manière montre pourquoi "nous avons restauré les systèmes" ne revient pas à dire "nous nous sommes rétablis". Chaque propriété exige sa propre réponse.

## Quand les trois propriétés se contredisent

Ces objectifs entrent souvent en conflit, et les nommer rend les compromis visibles.

- Chiffrer les données et exiger une authentification forte renforce la confidentialité, mais peut réduire la disponibilité si une clé est perdue ou si une connexion échoue.
- Conserver de nombreuses copies de sauvegarde améliore la disponibilité, mais multiplie les endroits où des données confidentielles pourraient fuiter.
- Verrouiller des enregistrements pour empêcher toute modification protège l'intégrité, mais peut faire obstacle aux mises à jour légitimes.

Il n'existe aucun classement universel. Les dossiers médicaux exigent une confidentialité et une intégrité fortes ; une page d'état publique exige avant tout la disponibilité, puisque son contenu est destiné à être vu par tous. FIPS 199 formalise cette pondération : les agences fédérales américaines évaluent séparément l'impact potentiel de la perte de chaque propriété, comme faible, modéré ou élevé, et choisissent leurs mesures de protection en conséquence.

## Au-delà des trois lettres

L'erreur la plus courante consiste à assimiler la sécurité à la confidentialité, c'est-à-dire garder les secrets secrets, en oubliant qu'un système corrompu ou inaccessible constitue tout autant une défaillance de sécurité. L'erreur inverse consiste à considérer la triade comme le tableau complet. C'est un socle, pas un cadre exhaustif.

Des modèles ultérieurs ajoutent des propriétés que les trois lettres ne couvrent pas entièrement. **L'authenticité** demande si une donnée ou un utilisateur est bien authentique. **La responsabilité** demande si les actions peuvent être rattachées à leur auteur (voir [responsabilité](../principles/accountability.md)). **La non-répudiation** demande si quelqu'un peut nier ultérieurement avoir envoyé un message ou approuvé une transaction. Certaines définitions, dont celle de la loi fédérale américaine, intègrent l'authenticité et la non-répudiation à l'intégrité ; d'autres les présentent comme des objectifs distincts.

## Références

- [NIST SP 800-12 Rev. 1, An Introduction to Information Security](https://csrc.nist.gov/pubs/sp/800/12/r1/final)
- [FIPS 199, Standards for Security Categorization of Federal Information and Information Systems](https://csrc.nist.gov/pubs/fips/199/final)
- [Glossaire NIST : Confidentiality](https://csrc.nist.gov/glossary/term/confidentiality)
- [Glossaire NIST : Integrity](https://csrc.nist.gov/glossary/term/integrity)
- [ISO/IEC 27000, Information security management systems overview and vocabulary](https://www.iso.org/standard/73906.html)
