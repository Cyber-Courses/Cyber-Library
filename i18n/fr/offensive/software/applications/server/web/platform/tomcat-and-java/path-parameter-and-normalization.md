---
title: "Paramètres de chemin de servlet et normalisation : traversal ..;/ et contournement de contrainte"
description: "Exploiter le traitement des chemins de servlet Java : le séparateur de paramètre de chemin ; et les différences de normalisation qui contournent les URL patterns des contraintes de sécurité et atteignent des servlets protégées et des applications de gestion."
keywords:
  - paramètres de chemin
  - ..;/
  - contrainte de sécurité de servlet
  - normalisation tomcat
  - contournement de contrôle d'accès
---

# Paramètres de chemin et normalisation

Les conteneurs de servlets Java traitent `;` comme le début d'un **paramètre de chemin** (un paramètre matriciel) au sein d'un segment de chemin, et ils normalisent les chemins selon leurs propres règles. Les contraintes de sécurité de `web.xml` (et les règles du proxy frontal) sont mises en correspondance avec des URL patterns, de sorte qu'un chemin qui se normalise différemment de chaque côté contourne la contrainte tout en se résolvant vers la servlet protégée.

## Le séparateur de paramètre de chemin ;

Un segment comme `admin;x=y` voit son `;x=y` retiré en tant que paramètre de chemin après les décisions de routage, de sorte que `..;/` est un segment de traversal que beaucoup de règles frontales et certaines vérifications de conteneur ne traitent pas comme `..` :

```
GET /app/..;/manager/html HTTP/1.1          # reach /manager past a /app-scoped rule
GET /;/admin/ HTTP/1.1
GET /app/js/..;/..;/admin HTTP/1.1
```

C'est une manière classique d'atteindre le [Tomcat Manager](manager-and-host-manager.md) ou d'autres contextes protégés lorsqu'un reverse proxy n'autorisait que `/app` : le proxy voit un chemin sous `/app`, le conteneur retire les paramètres de chemin et normalise vers `/manager`.

## Contournement de pattern de contrainte de sécurité

La `<security-constraint>` de `web.xml` met en correspondance des valeurs de `<url-pattern>`. Les désaccords entre le moteur de correspondance de pattern et le dispatcher permettent à des chemins forgés d'esquiver une contrainte :

- Caractères finaux ou supplémentaires (`/admin/`, `/admin%20`, `/admin.`) que le pattern de contrainte ne fait pas correspondre mais que le mapping de servlet fait correspondre.
- Différences de casse où la contrainte est sensible à la casse mais le mapping ne l'est pas (dépend du conteneur/OS).
- Séparateurs encodés (`%2e`, `%2f`, `%3b` pour `;`) que la vérification de contrainte ne décode pas mais que le conteneur décode.

## Exploitation

- Pour atteindre un contexte bloqué à l'edge ou par une contrainte, insérer `..;/` pour remonter et rediriger vers le contexte cible (`/manager/html`, `/host-manager/html`, servlets internes).
- Combiner avec le [désaccord de normalisation sur le reverse proxy](../reverse-proxy-and-edge/normalization-mismatch.md) : le proxy et Tomcat ne s'accordent pas sur ce que désigne `..;/`.
- Sonder les patterns de contrainte avec des variantes finales/encodées et comparer les réponses authentifiées et non authentifiées.

## Outils

- **curl --path-as-is** / Burp Intruder avec des payloads `;`/encodage.

## Références

- Apache Tomcat: security constraints, path parameter handling
- Orange Tsai: Java/servlet path confusion research
