---
title: "Configuration"
order: 11
description: "Périmètre offensif pour les plateformes de gestion de configuration d'entreprise : des systèmes qui poussent des logiciels, des scripts et des paramètres vers des terminaux gérés à grande échelle, et qui offrent donc à un attaquant l'exécution de code sur tout le parc une fois abusés."
keywords:
  - configuration management
  - SCCM
  - MECM
  - endpoint management
  - software deployment
---

# Configuration

Les plateformes de gestion de configuration existent pour pousser des logiciels, des scripts et des paramètres vers des milliers de machines gérées depuis une seule console. C'est aussi précisément ce que veut un attaquant : **l'exécution de code sur tout le parc** à partir d'un seul composant compromis. Ces systèmes bénéficient d'une confiance profonde, exécutent des agents en tant que SYSTEM sur chaque client, détiennent ou gèrent des identifiants, et sont généralement intégrés à Active Directory, de sorte qu'abuser de l'un revient souvent à posséder les terminaux du domaine.

Cette rubrique couvre la surface offensive de ces plateformes : comment les trouver, extraire les identifiants qu'elles stockent, prendre le contrôle de leur plan de contrôle et transformer leurs propres fonctionnalités de déploiement en exécution de code massive.

## Produits

Les plateformes natives Windows :

- **[SCCM / MECM](sccm/index.md)** : Microsoft Configuration Manager, la plateforme on-premises dominante et la surface d'attaque la plus riche du groupe.
- **[WSUS](wsus.md)** : Windows Server Update Services, détourné pour pousser une mise à jour malveillante qui s'exécute en tant que SYSTEM sur les clients gérés.

Les outils de gestion de configuration multiplateformes, chacun exécutant son agent ou sa connexion en tant que root sur chaque nœud géré :

- **[Ansible](ansible.md)** : push sans agent via SSH ; abuser du nœud de contrôle, des secrets Vault, des playbooks et d'AWX/Tower.
- **[SaltStack](saltstack.md)** : master et minions via ZeroMQ ; commander le parc depuis le master et exploiter le serveur de requêtes non authentifié.
- **[Puppet](puppet.md)** : modèle pull depuis le serveur Puppet ; injecter dans le dépôt de contrôle et piller Hiera et PuppetDB.
- **[Chef](chef.md)** : modèle pull piloté par les identifiants knife, les cookbooks, les data bags et les run lists.

## Périmètre

Microsoft Intune est un gestionnaire de terminaux hébergé dans le cloud, le pendant cloud de SCCM. Il est attaqué comme un service hébergé par un éditeur et relève de la surface d'identité et de gestion de terminaux cloud sous Online, pas ici.

## Références

- [SpecterOps : Misconfiguration Manager, la base de connaissances d'attaque et de défense SCCM](https://github.com/subat0mik/Misconfiguration-Manager)
- [Palo Alto Networks : SCCM, colonne vertébrale de l'entreprise ou vecteur d'attaque](https://www.paloaltonetworks.com/blog/security-operations/sccm-enterprise-backbone-or-attack-vector/)
