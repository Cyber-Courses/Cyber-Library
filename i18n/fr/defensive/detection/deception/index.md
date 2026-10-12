---
title: "Leurre de détection : pots de miel, leurres et tokens canari"
order: 4
description: "Comment la tromperie utilise pots de miel, leurres et tokens canari pour produire des signaux d'intrusion à forte confiance."
keywords:
  - tromperie cyber
  - pots de miel
  - systèmes leurres
  - tokens canari
  - alertes à forte confiance
  - détection d'intrusion
---

# Leurre

Le leurre est la pratique qui consiste à placer des pots de miel, des leurres et des canaris dans tout un environnement de sorte que toute interaction avec eux signale une activité probablement malveillante. Parce que les utilisateurs et systèmes légitimes n'ont aucune raison de toucher ces actifs implantés, une seule interaction devient un indicateur à forte confiance qui mérite une attention immédiate.

Au sein de la phase de détection, le leurre offre un type de signal différent de la plupart des analytiques. La détection conventionnelle tamise l'activité réelle pour en trouver la fraction suspecte, ce qui produit inévitablement des faux positifs. Le leurre inverse le problème : le leurre n'existe que pour être déclenché par quelqu'un qui explore là où il ne devrait pas, de sorte que l'alerte résultante comporte très peu de bruit.

En pratique, le leurre va des fausses informations d'identification et documents jusqu'à des hôtes et services leurres entiers qui imitent les systèmes de production. Les tokens canari intégrés dans des fichiers, des répertoires ou des clés d'API se déclenchent lorsqu'on y accède, révélant une reconnaissance ou un vol de données. Un leurre efficace est crédible et bien placé, se fondant dans l'environnement afin que les adversaires ne puissent pas facilement le distinguer des cibles réelles. Parce que les signaux sont si propres, le leurre se marie bien avec la réponse automatisée et avec la traque qui enquête sur la façon dont un intrus a atteint le leurre.

## Références

- MITRE Engage, Adversary Engagement framework
- NIST SP 800-160 Volume 2, Developing Cyber-Resilient Systems
