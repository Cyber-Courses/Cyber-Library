---
title: "Éradication en réponse à incident : retirer la présence de l'adversaire"
description: "Comment l'éradication retire la présence de l'adversaire et ferme le vecteur d'entrée pour qu'un incident ne puisse pas simplement reprendre."
keywords:
  - éradication d'incident
  - retrait de la présence adverse
  - fermeture du vecteur d'entrée
  - suppression des logiciels malveillants
  - suppression de la persistance
  - élimination de la menace
---

# Éradication

L'éradication est le retrait de la présence de l'adversaire dans l'environnement et la fermeture du vecteur qui lui a permis d'entrer. Là où le confinement maintient le problème en place, l'éradication l'élimine, en veillant à ce que, lorsque les systèmes reviennent en service, la menace ne reprenne pas simplement.

Au sein de la phase de réponse, l'éradication fait suite au confinement et précède la restauration. Elle dépend d'une investigation approfondie, car tout élément manqué (un point d'ancrage négligé, une porte dérobée, un identifiant compromis) peut permettre à l'adversaire de revenir. L'éradication relève donc autant de l'exhaustivité que de l'action : l'objectif est de ne laisser aucun chemin par lequel la même intrusion pourrait se poursuivre.

En pratique, l'éradication supprime les logiciels malveillants et l'outillage de l'attaquant, efface les comptes non autorisés et les mécanismes de persistance, révoque ou réinitialise les identifiants compromis, et remédie à la faiblesse qui a permis l'accès initial. Les systèmes fortement compromis sont souvent reconstruits à partir de sources connues et fiables plutôt que nettoyés sur place, afin d'être certain qu'il ne reste rien. Les intervenants vérifient que l'accès de l'adversaire a réellement disparu avant de passer à la restauration, en renforçant souvent la surveillance pour confirmer que l'environnement reste sain. Fermer le vecteur d'entrée, qu'il s'agisse d'une vulnérabilité, d'une mauvaise configuration ou d'un identifiant dérobé, fait de l'éradication une amélioration durable plutôt qu'un correctif temporaire.

## Références

- NIST SP 800-61, Computer Security Incident Handling Guide
- NIST SP 800-184, Guide for Cybersecurity Event Recovery
