---
title: "Injection d'argument"
description: "Une valeur utilisateur atterrit dans argv et le binaire appelé la ré-analyse comme une option plutôt que comme des données, sans shell, sans nouveau processus, et pourtant lecture/écriture de fichiers et exécution de code."
keywords:
  - injection d'argument
  - injection argv
  - injection d'option
  - injection de flag
  - manipulation de paramètre
---

# Argument

L'injection d'argument vise le cas sans shell : un binaire fixe est lancé avec un tableau d'arguments, de sorte que les métacaractères de shell sont inertes, mais une valeur contrôlée par l'utilisateur atterrit dans `argv` où le programme la ré-analyse comme une **option** plutôt que comme des données. Aucune *commande shell distincte* n'est introduite, et pourtant le binaire lancé est orienté vers la lecture de fichiers, l'écriture de fichiers ou le lancement de programmes auxiliaires qu'il prend nativement en charge (`curl -o`, `git -c`, `tar --checkpoint-action`, etc.).

## Pages

- **[Manipulation de paramètre](parameter-manipulation.md)** : Transformer une valeur utilisateur en argv pour qu'un binaire appelé l'analyse comme un flag, sans shell, plus les astuces de worst-fit et de caractères fullwidth et le contournement d'escapeshellarg/...

## Outils

- **[GTFOBins](https://gtfobins.github.io/)** : flags de lecture, d'écriture de fichiers et d'exécution de commandes par binaire.
- **[Burp Suite](https://portswigger.net/burp)** : Repeater et Intruder pour fuzzer un emplacement d'entrée avec des flags candidats.

## Références

- [PortSwigger Web Security Academy: OS command injection](https://portswigger.net/web-security/os-command-injection)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
