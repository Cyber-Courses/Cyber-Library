---
title: "Prototype pollution côté serveur dans Node.js vers contournement d'authentification et RCE"
description: "Du JSON non fiable fusionné en profondeur ou assigné récursivement laisse les clés __proto__ et constructor.prototype empoisonner Object.prototype, modifiant des valeurs par défaut partagées qui pilotent le contournement de logique et l'exécution de commande par chaîne de gadgets."
keywords:
  - prototype pollution
  - Node.js côté serveur
  - payload __proto__
  - constructor prototype
  - puits de deep merge
  - RCE par chaîne de gadgets
---

# Prototype pollution

La prototype pollution abuse du fait qu'en JavaScript chaque objet simple partage un unique prototype, `Object.prototype`. Lorsqu'une application copie des clés non fiables dans un objet avec un merge ou une assignation récursive, une clé nommée `__proto__` ou `constructor.prototype` ne crée pas une propriété ordinaire, elle remonte vers le prototype partagé et y écrit. Tout objet du processus qui ne définit pas cette propriété lui-même hérite désormais de la valeur de l'attaquant. Rien d'écrit par l'attaquant n'est évalué ; l'attaque modifie les valeurs par défaut dans lesquelles l'application lit, et l'impact dépend de l'endroit où ces valeurs par défaut sont consommées plus tard.

## Le puits

Le motif vulnérable est toute copie récursive de données contrôlées par l'attaquant dans un objet, où le chemin de clé est pris de l'entrée plutôt que d'une liste d'autorisation fixe. Les deep merges faits main, les anciens helpers `lodash` merge/set, les parseurs de query-string qui construisent des objets imbriqués, et les wrappers récursifs d'`Object.assign` en sont les sources habituelles.

```javascript
// recursive merge that honors attacker-chosen keys
function merge(target, source) {
  for (const key in source) {
    if (typeof source[key] === 'object' && source[key] !== null) {
      if (!target[key]) target[key] = {};
      merge(target[key], source[key]);   // recurses into __proto__
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

app.post('/profile', (req, res) => {
  const user = {};
  merge(user, req.body);                 // req.body is untrusted JSON
});
```

Lorsque la récursion descend dans une clé nommée `__proto__`, `target[key]` se résout en `Object.prototype`, et l'assignation interne écrit sur le prototype partagé au lieu d'écrire sur `user`.

## Le payload de pollution

Le corps de la requête porte une clé `__proto__` dont les propriétés imbriquées deviennent des valeurs par défaut globales :

```json
{"__proto__": {"polluted": "yes"}}
```

Après cette requête, tout `({}).polluted` ultérieur dans le processus renvoie `"yes"`. Le chemin `constructor.prototype` atteint le même prototype et est la variante à employer quand `__proto__` est filtré, parce que de nombreux filtres ne bloquent que la chaîne littérale `__proto__` :

```json
{"constructor": {"prototype": {"polluted": "yes"}}}
```

Les deux atterrissent sur `Object.prototype`. Une confirmation rapide que la pollution a pris, indépendante de tout comportement applicatif, consiste à définir une propriété qu'aucun objet ne définit et à la relire depuis un objet neuf ailleurs dans le flux.

```json
{"__proto__": {"isAdmin": true}}
```

## Escalade : contournement de logique et d'autorisation

La pollution seule n'est pas le gain ; elle modifie des valeurs par défaut partagées, et l'exploit consiste à choisir une propriété que l'application lit sans la définir. Le code qui contrôle un drapeau optionnel avec un test de véracité et n'assigne jamais de valeur par défaut est la cible classique :

```javascript
// elsewhere in the app, long after the merge
if (user.isAdmin) grantAdmin();          // user never set isAdmin
```

Comme `user` ne définit pas `isAdmin`, la recherche retombe sur le prototype empoisonné et renvoie `true`. Le même mécanisme défait les contrôles qui lisent des drapeaux de configuration, des feature gates ou des attributs de contrôle d'accès depuis des objets construits à partir d'une entrée partielle. Les propriétés qui conditionnent les décisions de sécurité (`isAdmin`, `role`, `authenticated`, `canEdit`), les propriétés qui contrôlent le parsing ou les limites, et les propriétés consommées par les bibliothèques en aval sont toutes des candidates. La valeur empoisonnée est partagée à l'échelle du processus, donc elle affecte aussi les requêtes d'autres utilisateurs jusqu'au redémarrage du processus, ce qui élargit une seule requête d'empoisonnement en un changement d'état global.

## Escalade : chaînes de gadgets vers l'exécution de commande

L'exécution de commande nécessite un gadget atteignable : un endroit où l'application ou une dépendance lit une propriété empoisonnée et la passe dans un puits dangereux. La pollution fournit la valeur ; le gadget fournit le chemin vers l'exécution. Le gadget côté serveur le plus cité est la gestion des options de `child_process`, où une option empoisonnée contrôle la façon dont une commande est lancée.

`child_process.spawn`, `exec` et `execSync` consultent un objet d'options pour `shell`, `env` et les réglages associés. Qu'un prototype empoisonné les atteigne dépend de la version : les versions actuelles de Node normalisent les options en un objet à prototype null en copiant les propriétés propres avant de lire `shell`/`env`, de sorte que les valeurs héritées d'`Object.prototype` ne transitent plus. Sur les versions plus anciennes de Node (et dans les bibliothèques qui lancent avec leur propre gestion d'options qui lit les propriétés héritées), les valeurs par défaut empoisonnées sont reprises lorsque l'appelant ne les a pas définies :

```json
{"__proto__": {"shell": "/proc/self/exe", "argv0": "node", "NODE_OPTIONS": "--require /proc/self/environ"}}
```

```json
{"__proto__": {"env": {"NODE_OPTIONS": "--require=/tmp/evil.js"}}}
```

Là où le gadget s'applique, un helper de spawn appelé sans `env` ni `shell` explicite reprend les valeurs par défaut empoisonnées, et `NODE_OPTIONS=--require` force Node à charger un module choisi par l'attaquant, qui exécute son code de niveau supérieur. Confirmez la version de Node de la cible et le chemin de spawn spécifique plutôt que de supposer que le gadget se déclenche partout. Les moteurs de template sont l'autre famille de gadgets classique : plusieurs (par exemple d'anciennes configurations d'EJS et de Pug/Jade) lisent des options de compilation telles que `outputFunctionName`, `escapeFunction`, ou un drapeau client/compileDebug depuis l'objet d'options, et une option empoisonnée est concaténée dans le source de la fonction générée, transformant le rendu en exécution de code.

```json
{"__proto__": {"outputFunctionName": "x;process.mainModule.require('child_process').execSync('id');x"}}
```

La chaîne est toujours en deux parties : un puits de pollution qui écrit le prototype, et un gadget distinct qui lit une propriété non définie dans un puits. Confirmez d'abord la pollution avec une propriété marqueur inoffensive, puis énumérez quelles bibliothèques de la cible lisent des objets d'options sans fixer de valeur par défaut à leurs champs, et faites correspondre le payload à l'un de ces gadgets. Là où aucun gadget n'est atteignable, le plafond réaliste est le contournement de logique et d'autorisation et le déni de service (par exemple polluer une propriété qui lève ensuite une exception ou force une branche inattendue à chaque requête).

## Outils

- **Burp Suite** : Repeater pour envoyer des payloads __proto__/constructor et confirmer les valeurs par défaut empoisonnées.
- **Server-Side Prototype Pollution Scanner** : extension Burp de PortSwigger détectant les puits de pollution côté serveur.
- Chasse manuelle de gadgets pour les propriétés d'options non définies lues par les bibliothèques.

## Références

- [OWASP: Prototype Pollution](https://cheatsheetseries.owasp.org/cheatsheets/Prototype_Pollution_Prevention_Cheat_Sheet.html)
- [PortSwigger: Server-side prototype pollution](https://portswigger.net/web-security/prototype-pollution/server-side)
- [PayloadsAllTheThings: Prototype Pollution](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Prototype%20Pollution)
- [Node.js: child_process](https://nodejs.org/api/child_process.html)
