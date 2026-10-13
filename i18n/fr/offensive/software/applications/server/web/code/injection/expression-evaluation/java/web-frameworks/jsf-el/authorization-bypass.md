---
title: "Contournement d'autorisation JSF EL : forcer les décisions d'accès via les attributs à portée"
description: "L'unified EL de JSF atteint les maps de portée session, requête et application, donc une expression #{...} injectée lit et écrase les attributs dont dépend un contrôle de rôle ou de drapeau."
keywords:
  - contournement d'autorisation JSF EL
  - sessionScope EL
  - objets implicites unified EL
  - contrôle d'accès JSF
  - injection EL
---

# Contournement d'autorisation

Les décisions d'accès JSF sont fréquemment exprimées en EL lui-même : un attribut `rendered`, une condition de navigation ou un composant sécurisé s'appuie sur `#{sessionScope.role == 'admin'}` ou `#{user.admin}`. Comme l'unified EL expose les maps d'attributs à portée sous forme d'objets implicites, une expression injectée lit ces mêmes valeurs et, là où le contexte d'évaluation est accessible en écriture, les fixe. La logique d'autorisation et l'attaquant partagent alors un seul espace de noms.

## Lecture des entrées de la décision

Les objets implicites `sessionScope`, `requestScope` et `applicationScope` sont des maps des attributs correspondants, et `param` contient les paramètres de requête. Une expression injectée lit tout ce que lit le contrôle :

```jsp
#{sessionScope.role}
#{sessionScope['isAdmin']}
#{applicationScope.featureFlags}
```

Lire la map de session énumère les noms et valeurs exacts des attributs auxquels un contrôle d'accès se compare, ce qui transforme un drapeau deviné en drapeau connu.

## Forcer la comparaison

Lorsque le contrôle compare un attribut à portée, une expression qui se résout à la valeur attendue le satisfait directement. Si le point d'injection est l'expression de garde elle-même, remplacez la comparaison par une vérité constante :

```jsp
#{true}
#{1==1}
```

Là où l'application évalue une expression de valeur face à un contexte EL accessible en écriture (un chemin `setValue`, ou un contexte EL 3.0 capable d'affectation), l'attribut à portée est écrasé de sorte qu'un contrôle ultérieur et distinct lise la valeur de l'attaquant :

```jsp
#{sessionScope.isAdmin = true}
```

## Atteindre la session via le contexte

Le contexte faces expose la requête et la session en direct, donc une expression navigue du contexte vers l'`HttpSession` et ses attributs même lorsque les maps de portée ne sont pas directement nommées par le template :

```jsp
#{facesContext.externalContext.sessionMap['role']}
#{facesContext.externalContext.request.getSession().getAttribute('user')}
```

Depuis le même `externalContext`, `getRequestParameterMap()` et la map de session permettent à l'expression de copier une valeur choisie par l'attaquant dans un attribut de session auquel le code en aval fait confiance, de sorte qu'un privilège détenu uniquement dans la requête est promu dans la session.

## Outils

- Tests manuels avec Burp Repeater ; payloads conçus par moteur.
- **Burp Intruder** : énumérer les noms d'attributs à portée auxquels un contrôle d'accès se compare.

## Références

- [Jakarta Expression Language Specification](https://jakarta.ee/specifications/expression-language/)
- [Jakarta Faces: ExternalContext](https://jakarta.ee/specifications/faces/)
