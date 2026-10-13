---
order: 3
title: "Abus de SOAP : dispatch, en-têtes et unmarshaling du corps"
description: "SOAP route les enveloppes XML par action et par corps vers des handlers d'opération. La surface d'attaque est la confusion de dispatch action contre corps, le traitement des en-têtes tel que WS-Security, et l'unmarshaling des paramètres du corps."
keywords:
  - injection SOAP
  - SOAPAction
  - WS-Security
  - corps SOAP
  - dispatch XML
---

# SOAP

SOAP enveloppe chaque appel dans une enveloppe XML avec un en-tête et un corps, et le serveur le route vers un handler d'opération en fonction d'un indice d'action et du premier élément du corps. La structure en couches, les métadonnées de transport, un en-tête `SOAPAction`, les en-têtes WS-*, et un corps typé, crée plusieurs endroits indépendants où le routage et l'autorisation peuvent diverger de ce que le handler exécute réellement.

## Les trois surfaces

- **[Usurpation de SOAPAction](soapaction-spoofing.md)** : routage ou autorisation calés sur l'en-tête `SOAPAction` tandis que le corps nomme une opération différente, de sorte qu'une requête autorisée pour une action en exécute une autre sur un endpoint partagé.
- **[Injection d'éléments d'en-tête](header-element-injection.md)** : WS-Addressing, WS-Security et les en-têtes SOAP personnalisés analysés depuis un message partiellement de confiance, où le rejeu, le stripping de signature ou des bogues de gestion de `mustUnderstand` résident dans le middleware.
- **[Injection de paramètres dans le corps](body-parameter-injection.md)** : enfants du corps et paramètres de document encapsulé construits à partir d'une entrée, où la confusion de type, les éléments dupliqués et les astuces `xsi:nil` modifient la façon dont le serveur unmarshale et autorise l'appel.

## Pourquoi l'enveloppe multiplie la surface

Un appel REST a un seul endroit qui dit ce qu'il est : la méthode et le chemin. Un appel SOAP le dit à plusieurs endroits, l'en-tête d'action, l'élément d'opération du corps et les en-têtes WS-*, et un framework peut faire confiance à l'un tout en exécutant depuis un autre. L'enveloppe est aussi analysée par une pile XML générale, de sorte que les préoccupations de markup brut de [traitement XML](../../markup/xml-processing/index.md) s'appliquent en dessous ; les pages ici se concentrent sur ce qui est spécifique à la liaison d'opération SOAP plutôt que sur l'analyse XML générique. Le test consiste à faire diverger l'en-tête d'action, l'opération du corps et les en-têtes de sécurité, et à voir lequel le serveur croit.

## Outils

- **Burp Suite** : interception et altération des enveloppes SOAP, des en-têtes et du SOAPAction.
- **SoapUI** : génération de requêtes à partir d'un WSDL et exercice des opérations.
- **Wsdler (extension Burp)** : analyse des définitions WSDL en requêtes Burp éditables.

## Références

- [OWASP: Testing for SOAP](https://owasp.org/www-project-web-security-testing-guide/)
- [W3C: SOAP Version 1.2](https://www.w3.org/TR/soap12-part1/)
