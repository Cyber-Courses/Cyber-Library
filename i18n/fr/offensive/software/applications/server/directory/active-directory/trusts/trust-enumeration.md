---
title: "Énumération des approbations : cartographier les approbations de domaine et de forêt"
order: 1
description: "Énumérer les relations d'approbation Active Directory : approbations intra-forêt et inter-forêts, leur direction et leur transitivité, et les attributs qui déterminent si une approbation est exploitable pour le mouvement latéral."
keywords:
  - trust enumeration
  - domain trust
  - forest trust
  - trustAttributes
  - SID filtering
---

# Énumération des approbations

Les approbations permettent aux principaux d'un domaine de s'authentifier auprès de ressources d'un autre. Pour un attaquant, elles sont la carte des endroits où une compromission peut se propager au-delà du domaine actuel, jusqu'à la racine de la forêt et vers d'autres forêts. L'énumération ici établit quelles approbations existent, leur direction et leur type, et si leur configuration permet les attaques au-delà des frontières.

## Lister les approbations

Les approbations sont stockées sous forme d'objets `trustedDomain` et sont lisibles par tout utilisateur du domaine :

```
# PowerView maps trusts and can walk them recursively
Get-DomainTrust
Get-DomainTrustMapping            # follow trusts across the environment
Get-ForestDomain -Forest example.local
```

```bash
# From a domain host / Linux
nltest /domain_trusts /all_trusts
nxc ldap <dc> -u user -p pass -M enum_trusts
ldapsearch ... '(objectClass=trustedDomain)' name trustDirection trustType trustAttributes
```

## Lire l'approbation

Trois attributs déterminent l'exploitabilité :

- **`trustDirection`** : entrant, sortant ou bidirectionnel. Les attaques suivent la direction de l'approbation (vous abusez d'une approbation qui fait confiance à *votre* domaine).
- **`trustAttributes`** : c'est l'attribut qui définit la topologie, via des drapeaux comme `WITHIN_FOREST` (une approbation intra-forêt parent-enfant ou tree-root), `FOREST_TRANSITIVE` (une approbation de forêt), et le fait que le **filtrage de SID** / la quarantaine (`TREAT_AS_EXTERNAL`, `QUARANTINED_DOMAIN`) soit activé ou non. Le filtrage de SID est le contrôle qui bloque l'injection de SID history à travers l'approbation, sa présence ou son absence détermine donc si l'escalade inter-forêts est viable.
- **`trustType`** : le *protocole et le type de partenaire* de l'approbation (une approbation Windows/AD up-level par opposition, par exemple, à un realm MIT Kerberos), et non la distinction parent-enfant contre externe contre forêt. Les approbations intra-forêt, externes et de forêt peuvent toutes rapporter le même type up-level, lisez donc la topologie à partir de `trustAttributes` ci-dessus, pas de `trustType`.

## Intra-forêt contre inter-forêts

La distinction est centrale :

- **Approbations intra-forêt** (parent-enfant, tree-root) partagent la chaîne `krbtgt` de la forêt et, surtout, ne filtrent **pas** le SID `Enterprise Admins`, de sorte que la compromission de n'importe quel domaine d'une forêt mène à la racine de la forêt. La forêt, et non le domaine, est la véritable frontière de sécurité.
- **Approbations inter-forêts** sont la frontière de sécurité ; le filtrage de SID est activé par défaut, l'escalade à travers elles requiert donc une configuration plus faible ou une primitive différente.

## Notes d'exploitation

- Cartographiez le graphe complet des approbations avant de planifier un déplacement ; `Get-DomainTrustMapping` et les arêtes d'approbation de BloodHound révèlent des domaines atteignables dont vous ignoriez peut-être l'existence.
- Notez si chaque approbation est transitive et si le filtrage de SID est activé, car ces deux faits conditionnent les attaques inter-domaines par SID history et par ticket traitées dans la section Approbations.
- Les foreign security principals (membres des groupes d'un domaine qui résident dans un autre domaine) sont un indicateur discret d'une relation d'approbation utilisable à examiner.

## Outils

- **PowerView `Get-DomainTrust` / `Get-DomainTrustMapping`** : découverte et parcours des approbations.
- **nltest /domain_trusts** : listage natif des approbations.
- **BloodHound** : arêtes d'approbation entre domaines et forêts.

## Références

- The Hacker Recipes: Trusts
- Microsoft: Active Directory trust types and attributes
