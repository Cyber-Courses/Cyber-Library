---
title: "Spoofing de sAMAccountName : l'escalade noPac"
order: 8
description: "Escalader d'un utilisateur de domaine standard vers administrateur de domaine en renommant un compte machine contrôlé pour correspondre à un contrôleur de domaine, en exploitant la façon dont le KDC résout les principals lors de l'échange TGS S4U2self (noPac)."
keywords:
  - sAMAccountName spoofing
  - noPac
  - sAMAccountName
  - S4U2self
  - machine account quota
---

# Spoofing de sAMAccountName

Cette escalade (couramment « noPac ») abuse de la façon dont le KDC recherche les comptes par nom. Si un compte machine est renommé pour que son `sAMAccountName` corresponde au nom d'un **contrôleur de domaine** (sans le `$` final), on peut amener le KDC à émettre un ticket de service comme si la demande provenait du DC. À partir de là, S4U2self produit un ticket usurpant l'identité d'un Domain Admin. Il part d'un utilisateur standard et, là où c'est non corrigé, aboutit à la compromission du domaine.

## La séquence

```bash
# One-shot (Impacket-based noPac / sam_the_admin)
noPac.py example.local/user:pass -dc-ip <dc> -impersonate Administrator -shell
```

En coulisses :

1. Créer un compte machine (le quota de comptes machine par défaut le permet), par exemple `EVIL$`.
2. Effacer ses SPN (un SPN bloquerait le renommage) et **renommer** son `sAMAccountName` au nom du DC sans `$`, par exemple `DC01`.
3. Demander un **TGT** pour ce nom. Le TGT est émis pour `DC01`.
4. Renommer le compte à son état initial (de sorte que `DC01` ne se résout plus vers lui). Désormais le TGT référence un principal que le KDC re-résout vers le **vrai DC**.
5. Utiliser **S4U2self** avec ce TGT pour demander un ticket de service en tant qu'`Administrator` vers le DC, obtenant un ticket qui usurpe l'identité d'un Domain Admin sur le DC.

## Pourquoi cela fonctionne

La faille est que le KDC, lors du traitement de S4U2self, résout le client par nom et, lorsque le principal d'origine n'existe plus, se rabat sur le compte DC correspondant, liant le ticket aux privilèges du DC. L'absence de validation du PAC dans les versions vulnérables est ce à quoi fait référence le nom « noPac ».

## Notes d'exploitation

- Nécessite seulement un **utilisateur de domaine standard** et un **quota de comptes machine** non nul (10 par défaut), ce qui en fait une primitive d'élévation de privilèges sévère là où les correctifs sont en retard.
- Le ticket obtenu usurpe l'identité d'un Domain Admin auprès du DC : utilisez-le pour [DCSync](../credentials/ntds-and-dcsync.md) ou pour une exécution directe.
- Si le quota est à 0, l'attaque nécessite un compte machine existant que vous contrôlez déjà pour le renommer.
- L'outillage gère la course entre le renommage et la demande ; le faire à la main exige un timing soigné des étapes 2 à 4.

## Outils

- **noPac.py / sam_the_admin.py** : exploitation automatisée de bout en bout.
- **Impacket** (`addcomputer.py`, `renameMachine.py`, `getST.py`) : les étapes manuelles.

## Références

- [noPac (sAMAccountName spoofing) exploit and writeup](https://github.com/cube0x0/noPac)
- [Impacket (addcomputer / renameMachine / getST)](https://github.com/fortra/impacket)
- [The Hacker Recipes: sAMAccountName spoofing](https://www.thehacker.recipes/ad/movement/kerberos/samaccountname-spoofing)
