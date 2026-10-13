---
title: "Contournement de disable_functions en PHP : atteindre l'exécution de commandes au-delà de la denylist"
description: "Obtenir l'exécution de commandes OS quand disable_functions de PHP bloque system/exec : alternatives non bloquées, LD_PRELOAD via un processus lancé, injection via mail, et FFI."
keywords:
  - contournement de disable_functions
  - LD_PRELOAD
  - putenv
  - FFI
  - mail
  - Chankro
---

# Contournement de disable functions en PHP

`disable_functions` est une denylist PHP de noms de fonctions. Elle est appliquée par l'interpréteur, c'est une denylist plutôt qu'une allowlist, et elle ne peut couvrir toutes les routes vers l'OS, de sorte qu'à partir d'un point d'appui PHP (webshell, primitive d'eval de code) il existe plusieurs moyens d'atteindre l'exécution de commandes malgré tout.

## Énumérer ce qui reste

Listez d'abord la restriction réelle et cherchez un sink d'exécution non bloqué :

```php
echo ini_get('disable_functions');
```

Les administrateurs bloquent couramment `system, exec, shell_exec, passthru, popen, proc_open` mais en manquent un parmi : `proc_open`, `popen`, `pcntl_exec`, `mail`, `mb_send_mail`, `imap_open`, `dl`, `putenv`, `error_log`, ou l'API FFI. Un seul survivant peut suffire.

## LD_PRELOAD via un sous-processus lancé

Le contournement générique le plus solide : faire lancer par PHP *n'importe quel* sous-processus, et le détourner avec un objet partagé malveillant via `LD_PRELOAD`. PHP fixe l'environnement avec `putenv`, et des fonctions comme `mail()`/`mb_send_mail()` forkent en interne `/usr/sbin/sendmail`. Pointez `LD_PRELOAD` vers votre `.so`, déclenchez le sous-processus, et le constructeur de votre bibliothèque exécute la commande hors des restrictions de PHP :

```php
putenv("LD_PRELOAD=/tmp/x.so");
mail("a","a","a","a");          // forks sendmail, loads x.so
```

L'objet partagé définit une fonction qui s'exécute au chargement et exécute la commande. **Chankro** automatise la construction du `.so` et du loader PHP :

```bash
python3 chankro.py --arch 64 --input cmd.sh --output shell.php --path /var/www/html
```

Cela fonctionne même quand toutes les fonctions `*exec*` sont désactivées, parce que l'exécution a lieu dans le processus C lancé, pas dans PHP.

## FFI

Si FFI est utilisable à l'exécution de la requête, appelez libc directement depuis PHP et contournez entièrement la denylist. La condition préalable est `ffi.enable=true`, et pas seulement l'extension chargée : le `ffi.enable=preload` par défaut restreint `FFI::cdef()` au CLI et au code préchargé et le bloque dans une requête web FPM/mod_php normale, donc un `FFI::cdef()` de webshell ne fonctionne que là où un opérateur a mis `ffi.enable=true` (rare en production).

```php
$ffi = FFI::cdef("int system(const char *command);");
$ffi->system("id");
```

## Autres routes

- **`imap_open`** : permettait historiquement l'injection de commande via son argument de mailbox (`-oProxyCommand`), une RCE en un appel quand IMAP est disponible.
- **`dl()`** : charger une extension PHP forgée si le chargement dynamique est permis.
- **Exploits d'interpréteur connus** : chaînes UAF/bug propres à une version (par exemple les exploits `GC`/`uaf` de PHP 7.x) qui désactivent la restriction de l'intérieur ; faites correspondre à la version PHP exacte.

## Notes d'exploitation

- Confirmez `open_basedir` séparément ; il peut vous empêcher d'écrire le `.so` ou de lire vos outils, auquel cas voyez d'abord [contournement de open_basedir](php-open-basedir-bypass.md).
- Vous avez besoin d'un chemin inscriptible pour le `.so`/payload (téléversements, `/tmp`, répertoire de session).
- Préférez la route LD_PRELOAD pour sa fiabilité sur les hôtes durcis ; FFI et `imap_open` sont plus rapides quand disponibles.

## Outils

- **Chankro** (LD_PRELOAD + sendmail), shells d'exploitation `disable_functions` préconstruits correspondant aux versions PHP.

## Références

- PHP manual: disable_functions, FFI
- Chankro project
