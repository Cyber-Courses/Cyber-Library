---
title: "Serveur : techniques offensives par type de service serveur"
order: 1
description: "La catégorie serveur organise les attaques selon le type de service testé, du web et des bases de données à l'annuaire et à la messagerie, car chaque classe de service a ses propres protocoles, primitives et schémas d'abus."
keywords:
  - server security
  - web application security
  - database attacks
  - directory services
---

# Serveur

La catégorie serveur couvre le travail offensif contre le code et les services qui s'exécutent sur une infrastructure contrôlée par l'opérateur, atteints à distance via les interfaces qu'ils exposent. Elle est organisée par type de service, car un frontal web, une base de données, un annuaire et un courtier de messages parlent chacun des protocoles différents et échouent de manières différentes.

## Pourquoi cette répartition

Les sous-catégories regroupent par classe de service, de sorte qu'une technique se trouve à côté du protocole et du modèle de données qu'elle abuse réellement :

- **[Virtualisation](virtualization/index.md)** : techniques offensives contre les hyperviseurs et leurs plans de gestion : évasions de VM de l'invité vers l'hôte, prise de contrôle du plan de gestion et vol de disques virtuels, à travers VMware, Hyper-V, KVM, Xen, Proxmox,...
- **[Web](web/index.md)** : applications exposées sur HTTP et les couches plateforme, runtime et code qui les servent. Le sous-arbre le plus vaste, et là où vivent la plupart des failles d'injection et de logique au niveau applicatif.
- **Base de données** : moteurs SQL et NoSQL, leurs langages de requête, leur authentification et leurs surfaces de gestion exposées.
- **Annuaire** : services d'identité et d'annuaire (LDAP, Active Directory), où dominent l'énumération et l'abus de privilèges.
- **Messagerie**, **Partage de fichiers**, **Accès distant**, **Supervision**, **Gestion de versions**, **Sécurité** : les services d'appui qu'exécute un environnement, chacun attaqué via son propre protocole et ses propres hypothèses de confiance.
- **Conteneurs**, **Virtualisation** : les couches d'isolation, où les évasions franchissent les frontières de charge de travail et de locataire. Les plateformes hébergées par un fournisseur (cloud, SaaS) constituent une catégorie distincte, [En ligne](../online/index.md).
- **Réseau** : les services réseau côté serveur en tant que cible à part entière, distincts du travail au niveau du médium dans la catégorie [réseau](../../../network/index.md) de premier niveau.

Répartir par type de service correspond à la façon dont un opérateur trie un hôte : énumérer ce qui écoute, identifier chaque service, puis mobiliser les techniques qui lui sont propres. Garder les classes séparées empêche qu'une technique de base de données soit classée à côté d'une technique de file de messages, car les langages de requête, les modèles d'authentification et les primitives d'abus n'ont rien en commun. Le sous-arbre web est subdivisé par couche (plateforme, runtime et code), car un seul service web combine les trois.

## Sous-thèmes

- **[Configuration](configuration/index.md)** : portée offensive des plateformes de gestion de configuration d'entreprise : systèmes qui poussent logiciels, scripts et paramètres vers des terminaux gérés à grande échelle, et...
- **[Conteneurs](containers/index.md)** : techniques offensives contre les charges de travail conteneurisées sur trois couches : l'évasion indépendante du runtime d'un conteneur vers son hôte, les runtimes de conteneurs et les plans de contrôle...
- **[Base de données](database/index.md)** : portée offensive des serveurs de bases de données atteints en tant que service : s'authentifier, énumérer et escalader à l'intérieur du moteur, exécuter du système d'exploitation...
- **[Annuaire](directory/index.md)** : techniques offensives contre les services d'annuaire : la surface d'attaque Active Directory (énumération, authentification et abus d'identifiants, ACL, relations de confiance,...
- **[Partage de fichiers](file-share/index.md)** : techniques offensives contre les services de partage et de transfert de fichiers : partages SMB, NFS et AFP, protocoles de transfert FTP, SFTP, TFTP et rsync, et accès aux fichiers par le web y compris WebDAV et le transfert de fichiers géré...
- **[Messagerie](messaging/index.md)** : portée offensive des serveurs de messagerie et de courrier d'entreprise : systèmes qui authentifient les utilisateurs, se tiennent sur le périmètre et détiennent à la fois les identifiants et la confiance...
- **[Réseau](network/index.md)** : portée offensive des services réseau et des interfaces RPC que les hôtes Windows et d'infrastructure exposent, où un seul service atteignable peut...
- **[Virtualisation](virtualization/index.md)** : techniques offensives contre les hyperviseurs et leurs plans de gestion : évasions de VM de l'invité vers l'hôte, prise de contrôle du plan de gestion et vol de disques virtuels, à travers VMware, Hyper-V, KVM, Xen, Proxmox,...
- **[Web](web/index.md)** : là où vivent les vulnérabilités web : la plateforme qui héberge le service, le runtime qui l'exécute et le code applicatif écrit par les développeurs.

## Références

- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [MITRE ATT&CK](https://attack.mitre.org/)
