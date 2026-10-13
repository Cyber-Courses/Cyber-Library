---
order: 3
title: "Injection SQL basée sur les erreurs dans Oracle"
description: "Divulguer les résultats de requêtes Oracle via des erreurs ORA qui renvoient l'entrée de l'attaquant, avec CTXSYS.DRITHSX.SN et UTL_INADDR, et les réserves liées à la version et aux ACL."
keywords:
  - injection SQL basée sur les erreurs
  - CTXSYS.DRITHSX.SN
  - UTL_INADDR
  - ORA-20000
  - ORA-29257
---

# Basée sur les erreurs

Lorsque l'application masque les lignes mais reflète les erreurs de la base de données, Oracle divulgue des données via des fonctions qui lèvent une erreur contenant leur argument. Deux sont fiables, chacune avec des réserves.

`CTXSYS.DRITHSX.SN` lève `ORA-20000` avec le texte fautif et est souvent appelable sans droits réseau particuliers (elle nécessite le composant Oracle Text, présent dans la plupart des installations) :

```sql
' AND 1=CTXSYS.DRITHSX.SN(1,(SELECT user FROM dual))-- 
```

L'erreur se lit `ORA-20000: Oracle Text error: DRG-11701: thesaurus SCOTT does not exist` (avec la valeur à la place du nom du thésaurus), divulguant le résultat de la sous-requête.

`UTL_INADDR.GET_HOST_NAME` lève `ORA-29257: host <value> unknown`, renvoyant la chaîne quand la résolution de noms échoue :

```sql
' AND 1=UTL_INADDR.GET_HOST_NAME((SELECT banner FROM v$version WHERE ROWNUM=1))-- 
```

Celle-ci est liée au réseau, de sorte qu'à partir de la 11g elle est conditionnée par une ACL fine sur `UTL_INADDR` ; sans l'attribution d'ACL, elle lève une erreur d'accès au lieu de celle qui divulgue. D'autres canaux cités dans les listes de payloads (erreurs d'analyse `XMLType`, `DBMS_XMLGEN`, `ORD_DICOM.GETMAPPINGXPATH`) varient selon la version et les composants installés.

Gardez la requête interne à une seule valeur avec `ROWNUM=1` ou un agrégat. Parce qu'elles lèvent des codes ORA distincts en cas de succès, les mêmes fonctions pilotent aussi l'extraction aveugle basée sur les erreurs lorsque seule la présence de l'erreur, et non son texte, est observable.

## Outils

- **sqlmap** : extraction basée sur les erreurs automatisée avec des payloads Oracle (`--dbms=Oracle --technique=E`).
- **ghauri** : alternative rapide avec une forte évasion de WAF.
- **sqlplus** (ou SQLcl) : le client Oracle pour rejouer et affiner les fonctions levant des erreurs.

## Références

- Oracle Database PL/SQL Packages and Types Reference : UTL_INADDR ; Oracle Text Reference
- OWASP Testing Guide : Testing for SQL Injection
