---
order: 1
title: "Usurpation de SOAPAction"
description: "Lorsque le routage ou l'autorisation est calé sur l'en-tête SOAPAction tandis que l'exécution lit l'opération du corps, une requête autorisée pour une action en exécute une autre sur un endpoint partagé."
keywords:
  - usurpation de SOAPAction
  - confusion de dispatch
  - routage d'opération
  - autorisation SOAP
  - endpoint partagé
---

# Usurpation de SOAPAction

Une requête SOAP annonce son opération à deux endroits : l'en-tête HTTP `SOAPAction` et le premier élément enfant du Body SOAP. Lorsqu'un serveur, une gateway ou un filtre de sécurité prend sa décision de routage ou d'autorisation à partir de l'en-tête, mais que le handler exécute l'opération nommée dans le corps, les deux peuvent être amenés à diverger. Une requête qui présente une action inoffensive dans l'en-tête et une opération sensible dans le corps est autorisée contre l'en-tête et exécutée depuis le corps.

## Le décalage

L'en-tête dit une chose :

```
POST /services/Endpoint HTTP/1.1
SOAPAction: "urn:GetPublicStatus"
Content-Type: text/xml
```

Le corps en dit une autre :

```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <DeleteUser xmlns="urn:AdminService">
      <id>104</id>
    </DeleteUser>
  </soap:Body>
</soap:Envelope>
```

Un filtre qui autorise `GetPublicStatus` et bloque `DeleteUser` inspecte l'en-tête, voit l'action permise et laisse passer la requête. La couche de dispatch lit ensuite le Body, trouve `DeleteUser` et l'exécute. La décision de contrôle d'accès et l'exécution ont utilisé des entrées différentes.

## Où cela s'applique

La faille apparaît partout où un routage basé sur l'action se place devant une exécution basée sur le corps : une politique WS ou une gateway XML qui match sur `SOAPAction`, un module d'autorisation calé sur la chaîne d'action, ou un reverse proxy qui route par en-tête tandis que l'application dispatche par corps. Elle est particulièrement courante sur les endpoints qui hébergent de nombreuses opérations derrière une seule URL, puisque l'en-tête est le moyen bon marché de les distinguer et qu'on lui fait confiance pour correspondre au corps.

## Confirmer la faille

Envoyez une requête dont le `SOAPAction` nomme une opération permise et peu privilégiée et dont le Body nomme une opération restreinte, et observez laquelle s'exécute. Si l'opération restreinte s'exécute, le serveur dispatche depuis le corps tandis que quelque chose en amont a autorisé depuis l'en-tête. Les variations incluent un `SOAPAction` vide ou omis (certaines piles retombent alors sur le corps, d'autres rejettent), et un en-tête qui nomme une action inexistante pour sonder si le routage échoue en mode ouvert vers le corps. Le correctif sous-jacent est d'exiger que l'en-tête d'action et l'opération du corps correspondent avant le dispatch, de sorte qu'un décalage est le signal à rechercher.

## Outils

- **Burp Repeater** : définition d'un SOAPAction permis tandis que le corps nomme une opération restreinte.
- **SoapUI** : construction de requêtes par opération à partir du WSDL à apparier contre des actions décalées.
- **Wsdler (extension Burp)** : énumération des opérations à partir du WSDL à cibler.

## Références

- [OWASP: Testing for SOAP](https://owasp.org/www-project-web-security-testing-guide/)
- [W3C: SOAP Version 1.2 Part 2 (SOAPAction)](https://www.w3.org/TR/soap12-part2/)
