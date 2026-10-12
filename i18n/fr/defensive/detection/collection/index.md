---
title: "Collecte de détection : rassembler la télémétrie et les journaux de sécurité"
order: 3
description: "Comment les défenseurs rassemblent la télémétrie et les journaux des terminaux, du réseau, des identités et du cloud pour rendre visible l'activité des adversaires."
keywords:
  - collecte de télémétrie de sécurité
  - collecte de journaux
  - télémétrie des terminaux
  - journaux réseau et cloud
  - sources de données de détection
  - visibilité
---

# Collecte

La collecte est la pratique qui consiste à rassembler la télémétrie et les journaux dont dépend la détection. Elle couvre les terminaux, le trafic réseau, les systèmes d'identité et d'authentification, et les plans de contrôle du cloud, en amenant les signaux bruts dans un endroit où ils peuvent être recherchés, corrélés et conservés. Sans une collecte fiable, le travail de détection ultérieur n'a rien sur quoi raisonner.

Au sein de la phase de détection, la collecte se situe à la fondation. La découverte de l'activité d'un adversaire n'est possible que si les bons événements sont capturés au départ, c'est pourquoi les défenseurs décident quelles sources comptent, quels champs conserver et combien de temps les stocker. Les lacunes de couverture deviennent des angles morts à travers lesquels les attaquants peuvent se déplacer sans être vus.

En pratique, la collecte implique le déploiement d'agents et de collecteurs, l'activation de la journalisation d'audit et de flux, la normalisation des formats et la centralisation des données dans un SIEM ou un data lake. Les équipes équilibrent l'exhaustivité avec le coût et le bruit, en priorisant les sources de grande valeur telles que la création de processus, les événements d'authentification, le DNS et l'activité des API du cloud. Une bonne collecte est délibérée et revue au fil du temps, et non simplement tout ce qui peut être journalisé.

## Références

- NIST SP 800-92, Guide to Computer Security Log Management
- MITRE ATT&CK, Data Sources
