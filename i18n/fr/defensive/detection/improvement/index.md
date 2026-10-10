---
title: "Amélioration de la détection : mesurer la couverture et régler dans la durée"
order: 7
description: "Comment l'amélioration de la détection mesure la couverture et règle les détections au fil du temps, souvent à l'aide de cadres tels que MITRE ATT&CK."
keywords:
  - amélioration de la détection
  - couverture de détection
  - couverture mitre attack
  - réglage de la détection
  - métriques et mesure
  - amélioration continue
---

# Amélioration

L'amélioration est l'effort continu pour mesurer dans quelle mesure la détection fonctionne et pour la rendre meilleure au fil du temps. Elle demande quels comportements d'adversaire le programme peut réellement voir, où se trouvent les lacunes et si les détections existantes sont toujours performantes, puis dirige l'effort vers les faiblesses qui comptent le plus.

Au sein de la phase de détection, l'amélioration est la boucle de rétroaction qui maintient le reste honnête. La collecte, l'analyse, les alertes et l'ingénierie bénéficient toutes de savoir où la couverture est mince et où le bruit est élevé. Sans mesure, une équipe ne peut pas dire si elle se renforce ou si elle ne fait qu'accumuler des règles.

En pratique, l'amélioration utilise souvent un cadre commun tel que MITRE ATT&CK pour cartographier les détections par rapport aux techniques et visualiser la couverture. Les équipes suivent des métriques comme le nombre de détections par tactique, les taux de faux positifs, le temps de détection et les résultats des traques et des incidents. Les constats issus d'incidents réels et d'exercices purple team révèlent l'activité manquée, qui devient de nouvelles détections ou des détections affinées. Le travail est continu : la couverture est revue à un rythme régulier, les priorités se déplacent avec le paysage des menaces, et le réglage équilibre la sensibilité face au bruit afin que l'amélioration soit mesurée, et non supposée.

## Références

- MITRE ATT&CK, guide de cartographie de la couverture
- NIST SP 800-55, Performance Measurement for Information Security
