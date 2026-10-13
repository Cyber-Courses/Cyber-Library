---
title: "Contournement d'autorisation JSP EL : forcer les décisions d'accès via les attributs à portée"
description: "L'unified EL de JSP atteint les maps de portée session, requête et application via pageContext, donc une expression ${...} injectée lit et compare les attributs dont dépend un contrôle de rôle ou de drapeau."
keywords:
  - contournement d'autorisation JSP EL
  - sessionScope EL
  - pageContext EL
  - contrôle d'accès JSP
  - injection EL
---

# Contournement d'autorisation

Les pages JSP conditionnent couramment le balisage et les actions en EL : un bloc conditionnel s'affiche sur `${sessionScope.role == 'admin'}`, un lien apparaît quand `${user.admin}`, un tag est ignoré sauf si `${requestScope.authorized}`. L'unified EL expose les maps de portée et le `pageContext` sous forme d'objets implicites, donc une expression injectée lit les mêmes attributs que le contrôle et, là où la valeur transite vers un attribut de session ou d'application, promeut une valeur choisie par l'attaquant dans la portée de confiance.

## Lecture des entrées de la décision

`sessionScope`, `requestScope` et `applicationScope` sont les maps d'attributs ; `param` et `header` portent l'entrée de la requête. Une expression injectée énumère ce à quoi le contrôle se compare :

```jsp
${sessionScope.role}
${sessionScope['isAdmin']}
${applicationScope.features}
```

Lire la map de session transforme un nom d'attribut deviné en nom confirmé et montre la valeur exacte attendue par une comparaison.

## Forcer la comparaison

Lorsque le point d'injection est l'expression de garde, résolvez-la à une vérité constante pour que la branche gardée s'exécute toujours :

```jsp
${true}
${1==1}
${'admin' == 'admin'}
```

## Atteindre la session via pageContext

`pageContext` expose la requête, la réponse et la session en direct, donc une expression navigue depuis lui vers l'`HttpSession` et ses attributs même lorsque les maps de portée ne sont pas nommées par le template environnant :

```jsp
${pageContext.session.getAttribute('role')}
${pageContext.request.getSession().getAttribute('user')}
${pageContext.request.isUserInRole('admin')}
```

Depuis `pageContext.request`, l'expression lit aussi `getParameter(...)` et `getRemoteUser()`, de sorte qu'une valeur d'accès que l'application copie de la requête vers un attribut de session est observée, et la comparaison qui lui fait ensuite confiance est comprise avant d'être ciblée.

## Outils

- Tests manuels avec Burp Repeater ; payloads conçus par moteur.
- **Burp Intruder** : énumérer les noms d'attributs des maps de portée auxquels une garde se compare.

## Références

- [Jakarta Expression Language Specification](https://jakarta.ee/specifications/expression-language/)
- [Jakarta Pages: Implicit EL Objects](https://jakarta.ee/specifications/pages/)
