---
title: "BloodHound : collecter et cartographier les chemins d'attaque Active Directory"
order: 11
description: "Utilisation de BloodHound pour collecter les objets, sessions, ACL et relations d'approbation d'Active Directory avec SharpHound ou bloodhound-python, puis analyse du graphe pour trouver les chemins d'élévation de privilèges vers des cibles à forte valeur."
keywords:
  - BloodHound
  - SharpHound
  - bloodhound-python
  - chemin d'attaque
  - cypher
---

# BloodHound

BloodHound transforme l'annuaire en un graphe de principaux et des relations entre eux (appartenance à des groupes, sessions, droits d'ACL, délégation, relations d'approbation) puis répond à la seule question qui compte : quel est le chemin le plus court entre ce que je contrôle et ce que je veux. Il remplace les suppositions par un itinéraire calculé, et c'est la première étape d'analyse standard dès que vous pouvez lire le domaine.

## Collecte

Les collecteurs rassemblent les données ; l'interface graphique les analyse. Collectez avec ce qui correspond à votre position :

```bash
# From a non-domain Linux box, over the network with any domain creds
bloodhound-python -u user -p 'Password1' -d example.local -ns <dc-ip> -c All

# From a domain-joined Windows host
SharpHound.exe -c All
# or the PowerShell collector
Invoke-BloodHound -CollectionMethod All
```

`-c All` (ou `All,GPOLocalGroup`) rassemble les objets, l'appartenance aux groupes, les ACL, les relations d'approbation et les sessions. **La collecte des sessions et des admins locaux** (qui est connecté où, qui est admin local sur quoi) sont les parties qui révèlent les chemins de déplacement latéral, mais elles nécessitent d'interroger chaque hôte et sont plus bruyantes ; `DCOnly` est une alternative discrète qui récupère tout ce qui est lisible depuis le DC sans toucher aux postes de travail.

## Analyse

Chargez le JSON résultant (ou connectez le collecteur à la base de données BloodHound CE), marquez ce que vous possédez comme Owned, marquez les cibles comme High Value, et lancez les requêtes de chemins :

- **Chemins les plus courts vers Domain Admins** depuis les principaux possédés.
- **Requêtes prédéfinies** : utilisateurs Kerberoastable, utilisateurs AS-REP roastable, systèmes en délégation non contrainte, principaux disposant de droits DCSync, ACL dangereuses.
- **Cypher personnalisé** pour des recherches précises, par exemple les principaux disposant de `GenericAll` sur un groupe privilégié :

```cypher
MATCH p=(n)-[:GenericAll]->(g:Group) WHERE g.highvalue=true RETURN p
```

Chaque arête que BloodHound dessine correspond à une technique concrète dans les sections qui suivent : une arête `AdminTo` est une opportunité d'exécution latérale (pass-the-hash pour exécuter du code), une arête `GenericAll`/`WriteDacl` est un abus de DACL, une arête `HasSession` est une opportunité de vidage d'identifiants sur cet hôte, et une arête `AllowedToDelegate` est un chemin de délégation Kerberos.

## Notes d'exploitation

- Collectez depuis la position la moins privilégiée qui fonctionne ; un simple utilisateur de domaine peu privilégié peut généralement collecter les objets, les ACL et les relations d'approbation (la structure à forte valeur) même sans données de session.
- Les données de session sont sensibles au temps, recollectez-les donc avant de planifier un déplacement latéral plutôt que de vous fier à un instantané périmé.
- Le graphe vous indique le chemin ; les pages par arête vous indiquent comment parcourir chaque étape.

## Outils

- **BloodHound CE** : l'interface graphique et les requêtes de chemins.
- **SharpHound** : le collecteur Windows/.NET.
- **bloodhound-python** : le collecteur réseau pour les opérateurs non-Windows.

## Références

- [SpecterOps: BloodHound Community Edition docs](https://bloodhound.specterops.io/)
- [BloodHound (SpecterOps, GitHub)](https://github.com/SpecterOps/BloodHound)
- [BloodHound.py (dirkjanm): Python collector](https://github.com/dirkjanm/BloodHound.py)
- [SpecterOps: An ACE Up the Sleeve (AD DACL paths)](https://specterops.io/wp-content/uploads/sites/3/2022/06/an_ace_up_the_sleeve.pdf)
