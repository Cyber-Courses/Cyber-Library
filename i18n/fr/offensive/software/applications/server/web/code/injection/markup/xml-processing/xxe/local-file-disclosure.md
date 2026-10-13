---
title: "Divulgation de fichier local via XXE en bande"
description: "Une entité SYSTEM lit un fichier local et l'application reflète la valeur étendue dans sa réponse, renvoyant directement le contenu du fichier."
keywords:
  - lecture de fichier XXE
  - entité SYSTEM
  - divulgation de fichier
  - php filter base64
  - XXE en bande
---

# Divulgation de fichier local

Le XXE classique est en bande : déclarez une **entité générale** externe qui pointe vers un fichier local, référencez-la à un endroit que l'application renvoie, et lisez le fichier directement dans la réponse HTTP. Cela fonctionne dès que le parseur résout les entités externes et qu'une valeur parsée est reflétée vers le client, par exemple un message d'erreur, un résultat de recherche ou un champ rendu dans une page de confirmation.

## Lecture en bande

Définissez l'entité dans le sous-ensemble interne (la DTD entre crochets) et référencez-la dans un élément dont l'application renvoie le texte :

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```

Quand le parseur étend `&xxe;`, la valeur `productId` devient le contenu de `/etc/passwd`, et l'application le renvoie. Le schéma `file://` lit tout chemin que le compte de service peut ouvrir : `file:///etc/passwd`, `file:///etc/hostname`, `file:///proc/self/environ`, `file:///home/user/.ssh/id_rsa`, ou un chemin Windows tel que `file:///c:/windows/win.ini`.

Pour trouver le champ injectable, sondez avec une entité **externe**, pas une interne. Une entité de remplacement interne telle que `<!ENTITY test "INJECTED">` s'étend même quand le chargement d'entités externes est désactivé, donc la voir renvoyée ne prouve que le traitement des entités, pas que les lectures `file://` fonctionnent, un faux positif sur un parseur durci. Pointez plutôt la sonde vers un fichier qui existe de façon fiable :

```xml
<!DOCTYPE foo [ <!ENTITY test SYSTEM "file:///etc/hostname"> ]>
<stockCheck><productId>&test;</productId></stockCheck>
```

Si le hostname revient dans la réponse, le parseur résout les entités `file://` externes via ce champ et la lecture complète ci-dessus fonctionnera.

## Fichiers qui cassent le parseur

De nombreux fichiers intéressants ne sont pas des données caractère bien formées. XML interdit les `<`, `&` bruts et certains octets de contrôle dans le contenu, donc lire un fichier source, une configuration avec des caractères `<` ou tout blob binaire fait avorter le parseur avec une erreur fatale avant qu'il ne renvoie quoi que ce soit. Sur les cibles PHP, le wrapper `php://filter` résout cela en transformant la ressource avant qu'elle n'atteigne le parseur :

```xml
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM
    "php://filter/convert.base64-encode/resource=/var/www/html/config.php">
]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```

Le fichier est encodé en base64, donc il ne contient que `[A-Za-z0-9+/=]` et se parse toujours proprement. Décodez la chaîne reflétée pour récupérer les octets originaux, ce qui en fait la façon standard d'extraire du source PHP, des credentials de base de données et des fichiers binaires via un canal en bande. Enchaînez les filtres pour gérer les encodages délicats, par exemple `php://filter/read=convert.base64-encode/resource=index.php`.

D'autres schémas étendent la portée selon la plateforme et les extensions installées : `expect://id` pour l'exécution de commandes là où le wrapper expect est chargé, `data://` pour injecter en ligne une charge utile forgée, et `netdoc://` sur certaines piles Java comme lecteur de fichier alternatif. Quand le contenu du fichier n'est pas reflété du tout, basculez vers un canal hors bande ou basé sur les erreurs.

## Outils

- **XXEinjector** : automatisation de la récupération de fichier en bande via XXE.
- **Burp Suite** : forge et itération de charges utiles d'entités SYSTEM dans Repeater.
- Tests manuels avec des charges utiles `php://filter` base64 pour les fichiers non-XML.

## Références

- [OWASP: XML External Entity (XXE) Processing](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)
- [PayloadsAllTheThings: XXE Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)
