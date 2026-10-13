---
title: "Contournement de known_hosts : vaincre le trust-on-first-use de SSH"
description: "La protection de SSH contre une clé d'hôte substituée est l'épinglage dans known_hosts et l'avertissement de changement de clé d'hôte du client. Cette protection échoue à la première connexion (aucune clé épinglée), lorsque les clients sont configurés avec StrictHostKeyChecking no, lorsque les utilisateurs acceptent l'avertissement, et lorsque known_hosts peut être modifié, chacun permettant à la clé d'hôte d'un attaquant d'être approuvée et activant le machine-in-the-middle."
keywords:
  - known_hosts
  - stricthostkeychecking
  - confiance à la première utilisation
  - avertissement de clé d'hôte
  - tofu
---

# Contournement de known_hosts

L'épinglage dans `known_hosts` plus l'avertissement bruyant sur une clé d'hôte changée est ce qui empêche un attaquant de substituer sa propre clé. Ce modèle de confiance a des lacunes qu'un attaquant exploite pour faire approuver sa clé d'hôte. À une toute première connexion, il n'y a aucune clé épinglée, donc le trust-on-first-use accepte ce qui est présenté. De nombreux clients et automatisations fixent `StrictHostKeyChecking no` (ou `accept-new`), acceptant silencieusement les nouvelles clés. Les utilisateurs habitués à l'avertissement tapent `yes`. Et lorsqu'un attaquant peut écrire le `known_hosts` d'une victime (via un home accessible en écriture sur NFS/SMB, ou un pied-à-terre antérieur), il pré-épingle sa propre clé pour qu'aucun avertissement n'apparaisse jamais.

```bash
# conditions that let a substituted key be trusted:
grep -ri 'StrictHostKeyChecking' /etc/ssh/ssh_config ~/.ssh/config    # "no"/"accept-new" => silent accept
# first-connection TOFU: a target that has never connected has no pin to violate
# pre-poison a victim's known_hosts where you can write it (attacker host key for the target)
ssh-keyscan -H <target> 2>/dev/null   # shows the format; an attacker writes THEIR key as the target's
```

## Notes d'exploitation

- Les gains les plus faciles sont la configuration et le comportement : `StrictHostKeyChecking no` dans une config de client/automatisation signifie qu'une clé d'hôte de MITM est acceptée silencieusement, et l'automatisation (CI, scripts) fixe très couramment cela.
- Le trust-on-first-use est exploitable chaque fois que le client ne s'est pas encore connecté à la cible ; être sur le chemin de cette première connexion la capture avec un seul avertissement ponctuel (ou aucun, si la vérification stricte est désactivée).
- Écrire le `known_hosts` d'une victime pour pré-épingler la clé de l'attaquant supprime entièrement l'avertissement ; combinez avec une primitive d'écriture dans le répertoire personnel (usurpation d'UID NFS, partage accessible en écriture).
- Une vérification de clé d'hôte contournée est l'élément habilitant du [SSH MITM](ssh-mitm.md) ; à elle seule, c'est la défaillance de confiance qui rend l'interception possible.

## Références

- [OpenSSH: StrictHostKeyChecking](https://man.openbsd.org/ssh_config#StrictHostKeyChecking)
- [HackTricks: SSH known_hosts](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
