---
title: "Data Factory : pipelines et linked services sous l'identité de la factory"
order: 3
description: "Abuser d'Azure Data Factory : pipelines et linked services qui s'exécutent sous l'identité managée de la factory, et secrets de connexion stockés."
keywords:
  - Data Factory
  - pipeline
  - linked service
  - identité managée
  - integration runtime
---

# Data Factory

Azure Data Factory exécute des pipelines de données, et un pipeline s'exécute sous l'**identité managée** de la factory. Un principal capable de créer et de déclencher un pipeline exécute des activités (Web, Azure Function ou une activité personnalisée) sous cette identité, ce qui transforme le contrôle d'une data factory en un jeton pour tout ce qui est accordé à l'identité de la factory. Les linked services stockent aussi des secrets de connexion qui peuvent être relus.

## Exécuter un pipeline sous l'identité de la factory

```bash
az datafactory list -g <rg> -o table
# author a pipeline with a Web activity that calls ARM or Graph as the factory MI,
# or that hits IMDS-style token endpoints, then trigger it
az datafactory pipeline create-run --factory-name <df> -g <rg> --name <pipeline>
```

## Piller les linked services

```bash
# linked services hold connection strings and credential references
az datafactory linked-service list --factory-name <df> -g <rg>
```

## Notes d'exploitation

- L'identité assignée par le système de la factory se voit fréquemment accorder un accès à Storage ou Key Vault pour que les pipelines puissent lire des données ; une activité Web qui appelle ces API hérite de cet accès.
- Un integration runtime auto-hébergé s'exécute sur une VM que vous pourriez atteindre, et détient des identifiants pour des sources on-prem.
- Les exécutions de pipeline sont un canal d'exécution discret : elles ressemblent à un mouvement de données normal, pas à du calcul interactif.

## Outils

- **az cli** (`az datafactory pipeline create-run`, `az datafactory linked-service list`).
- **MicroBurst** : énumère les data factories et les secrets des linked services.

## Références

- [Microsoft : identité managée Data Factory](https://learn.microsoft.com/azure/data-factory/data-factory-service-identity)
- [HackTricks Cloud : Azure Data Factory](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [SpecterOps : chemins d'attaque par identité managée Azure (BARK)](https://github.com/BloodHoundAD/BARK)
