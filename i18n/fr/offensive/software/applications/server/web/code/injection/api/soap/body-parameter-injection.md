---
order: 2
title: "Injection de paramètres dans le corps SOAP"
description: "Les enfants du corps et les paramètres de document encapsulé construits à partir d'une entrée permettent à la confusion de type, aux éléments dupliqués et aux astuces xsi:nil de modifier la façon dont le serveur désérialise et autorise un appel SOAP."
keywords:
  - injection dans le corps SOAP
  - xsi:nil
  - élément dupliqué
  - confusion de désérialisation
  - wrapped literal
---

# Injection de paramètres dans le corps

Le Body SOAP contient les paramètres de l'opération sous forme d'éléments XML, que le serveur désérialise en objets typés avant l'exécution de la logique métier. Lorsque ces éléments sont construits à partir d'une entrée partiellement contrôlée, ou lorsque le désérialiseur est laxiste sur la structure, un attaquant modifie la forme des paramètres de manières que le schéma et la logique d'autorisation n'avaient pas anticipées : annuler un champ, dupliquer un élément ou confondre un type de sorte que l'objet désérialisé diffère de ce qui a été validé.

## Éléments dupliqués

XML autorise un élément de paramètre à apparaître plus d'une fois, et les unmarshalers divergent sur celui qui l'emporte (le premier, le dernier, ou une erreur). Si une étape de validation ou d'autorisation lit une occurrence et que la logique métier en lit une autre, la requête est vérifiée contre une valeur différente de celle avec laquelle elle s'exécute :

```xml
<soap:Body>
  <Transfer xmlns="urn:Bank">
    <amount>1</amount>
    <toAccount>self</toAccount>
    <toAccount>victim</toAccount>   <!-- which one does the handler use? -->
  </Transfer>
</soap:Body>
```

## xsi:nil et champs manquants

L'attribut `xsi:nil="true"` marque un élément comme null même s'il est présent. Lorsqu'un champ est censé porter une contrainte (un propriétaire, un tenant, un statut), l'annuler peut supprimer une vérification qui ne s'exécute que lorsque le champ est renseigné, ou contraindre l'objet vers une valeur par défaut que la logique traite comme privilégiée :

```xml
<GetDocument xmlns="urn:Docs" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <documentId>900</documentId>
  <ownerId xsi:nil="true"/>   <!-- bypass an owner-scoped filter that skips null owners -->
</GetDocument>
```

## Confusion de type dans la désérialisation

Lorsque le type d'un élément est inféré ou faiblement lié (un paramètre wrapped document/literal, une surcharge `xsi:type`, ou un champ désérialisé dans un type de base large), fournir un type ou une structure inattendus peut faire différer l'objet désérialisé de celui validé par le schéma. Un champ validé comme simple chaîne peut se désérialiser dans un type complexe dont les membres imbriqués atteignent du code que le cas simple n'atteignait jamais.

## Confirmer la faille

Envoyez des enveloppes structurellement valides qui mettent le désérialiseur sous pression : dupliquez un élément pertinent pour la sécurité avec des valeurs contradictoires, marquez un champ de contrainte `xsi:nil`, et surchargez un type avec `xsi:type`. Une réponse qui reflète le second doublon, honore le nil en sautant une vérification, ou traite le type surchargé montre que le désérialiseur et le validateur ont vu des structures différentes. La validation stricte par schéma avant la logique métier ferme cela, de sorte que c'est la permissivité dans la façon dont le corps est analysé que le test sonde. Les préoccupations d'analyse XML brute telles que l'expansion d'entités résident dans [traitement XML](../../markup/xml-processing/index.md) ; cette page porte sur la façon dont les paramètres unmarshalés pilotent l'opération.

## Outils

- **Burp Repeater** : élaboration d'éléments dupliqués, de xsi:nil et de surcharges xsi:type dans le corps.
- **SoapUI** : génération d'enveloppes valides à partir du WSDL pour les muter.
- **Wsdler (extension Burp)** : analyse d'un WSDL en requêtes éditables dans Burp.

## Références

- [OWASP: Testing for SOAP](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
