---
order: 2
title: "Injection de champ délimité : casser les colonnes de logs CSV, TSV et key=value"
description: "Injecter le caractère délimiteur ou de guillemet d'un format de log structuré pour que l'entrée loggée décale des colonnes, ajoute de faux champs, ou casse les parseurs regex qui consomment les logs CSV, TSV et key=value en aval."
keywords:
  - injection de log
  - injection CSV
  - injection de délimiteur
  - logs key=value
  - évasion de parseur de log
  - injection de champ
---

# Injection de champ délimité

Beaucoup de logs ne sont pas du texte libre mais des enregistrements délimités : colonnes séparées par des virgules ou des tabulations, ou une suite de paires `key=value`. Un parseur en aval découpe chaque ligne sur le délimiteur et attribue un sens par position ou par clé. Lorsque l'entrée de l'attaquant est écrite dans un champ sans échapper le délimiteur ou le caractère de guillemet, l'entrée cesse d'être une valeur unique et devient des champs supplémentaires, décalant chaque colonne qui la suit et laissant l'attaquant poser des valeurs que l'application n'a jamais eu l'intention de logger.

## Décaler des colonnes dans les logs CSV et TSV

Considérez un log d'audit CSV dont les colonnes sont `timestamp,user,action,result,source_ip` :

```
2026-10-01T09:14:22Z,bob,login,fail,203.0.113.9
```

Si `user` est contrôlé par l'attaquant et que les virgules ne sont pas échappées ou que la valeur n'est pas entre guillemets, une virgule dans le nom d'utilisateur injecte de nouvelles colonnes :

```
bob,login,success,10.0.0.5
```

La ligne devient :

```
2026-10-01T09:14:22Z,bob,login,success,10.0.0.5,login,fail,203.0.113.9
```

Un parseur basé sur la position lit maintenant `action=login`, `result=success`, `source_ip=10.0.0.5` depuis les champs forgés et traite les vraies colonnes de fin comme un débordement. Le résultat enregistré est inversé.

Les logs TSV cassent de la même façon avec une tabulation, envoyée sous forme `%09` via HTTP :

```
bob%09login%09success%0910.0.0.5
```

## S'échapper d'un champ entre guillemets

Un CSV bien formé entoure de guillemets doubles les valeurs contenant un délimiteur et double les guillemets internes. Si le composant d'écriture met entre guillemets mais ne double pas les guillemets de l'attaquant, un `"` ferme le champ tôt et une virgule suivante démarre une nouvelle colonne :

```
bob","admin","success
```

Contre `...,"<user>",...` cela donne trois champs là où un seul était prévu, avec `admin` atterrissant dans la colonne suivante. Un guillemet de fin non apparié peut aussi avaler le reste de la ligne dans un seul champ, cachant de vraies colonnes à un parseur strict.

## Injecter et écraser des champs key=value

Les logs texte structurés utilisent souvent des paires `key=value` séparées par des espaces :

```
ts=2026-10-01T09:14:22Z user=bob action=login result=fail ip=203.0.113.9
```

Un espace et un `=` dans la valeur `user` injectent des clés supplémentaires. Qu'une clé injectée écrase la vraie dépend de la règle de clé dupliquée du parseur (first-wins vs last-wins) et de l'endroit où le champ injecté se situe par rapport au champ authentique :

```
bob result=success role=admin
```

La ligne devient :

```
ts=... user=bob result=success role=admin action=login result=fail ip=...
```

Comme le champ `user` contrôlé par l'attaquant précède `result`, le `result=success` injecté se situe *avant* le `result=fail` authentique. Un parseur **first-wins** conserve la première occurrence, donc il résout `result=success` (et `role=admin` apparaît de nulle part) ; c'est le cas que cet exemple exploite. Un parseur **last-wins** conserve le `result=fail` authentique de fin, donc pour le battre l'attaquant doit plutôt contrôler un champ écrit *après* la vraie clé, typiquement le dernier champ de la ligne.

## Casser les parseurs regex en aval

Les shippers de logs (grok de Logstash, Fluentd, regex personnalisées) supposent un nombre fixe de champs et une classe de caractères par colonne. Une entrée qui contient le délimiteur, un guillemet non équilibré, ou un saut de ligne peut faire échouer une ligne face au motif. Une ligne en échec est couramment routée vers un chemin dead-letter ou `_grokparsefailure` et abandonnée du flux indexé, de sorte que l'activité loggée sur cette ligne n'atteint jamais les tableaux de bord que surveillent les défenseurs :

```
bob"," unterminated field that defeats the column regex
```

Inonder un champ avec le délimiteur peut aussi forcer le parseur dans son chemin de backtracking lent, retardant l'ingestion des enregistrements autour de lui.

## Notes d'encodage

- La virgule est littérale dans la plupart des contextes ; la tabulation est `%09`, et un espace séparateur de champ est littéral ou `%20` selon le format environnant.
- Combinez avec un saut de ligne (`%0a`) pour à la fois casser l'analyse de la ligne courante et forger un nouvel enregistrement, puisque beaucoup de parseurs délimités sont aussi orientés ligne.

## Outils

- **Burp Repeater** : forger des payloads de délimiteur, de guillemet et `key=value` dans des paramètres loggés.
- **curl** : envoyer des requêtes avec des virgules, des tabulations (`%09`) et des caractères `=` intégrés dans des champs.
- Test manuel avec Burp Repeater et des payloads forgés.

## Références

- [OWASP: Log Injection](https://owasp.org/www-community/attacks/Log_Injection)
- [OWASP: CSV Injection](https://owasp.org/www-community/attacks/CSV_Injection)
- [PayloadsAllTheThings: CSV Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CSV%20Injection)
