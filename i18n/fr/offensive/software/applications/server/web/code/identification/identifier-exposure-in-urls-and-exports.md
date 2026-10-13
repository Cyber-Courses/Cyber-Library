---
title: "Exposition d'identifiants : IDs séquentiels, API trop bavardes et exports qui fuitent"
description: "Comment les applications fournissent aux attaquants les identifiants dont ils ont besoin, via des IDs séquentiels ou prévisibles dans les URLs, des réponses d'API trop bavardes, des exports avec des champs cachés, et des endpoints de profil public et d'autocomplétion."
keywords:
  - exposition d'identifiants
  - IDs séquentiels
  - reconnaissance IDOR
  - exposition excessive de données
  - énumération d'utilisateurs
  - export de données
---

# Exposition d'identifiants dans les URLs et les exports

Les applications émettent constamment des identifiants : IDs utilisateur, numéros de compte, références de commande, clés d'objets internes. Lorsque ces valeurs sont prévisibles ou divulguées plus largement que l'interface ne le laisse entendre, elles deviennent la matière première de l'énumération et des attaques de contrôle d'accès. Cette page porte sur la *collecte* d'identifiants, l'étape qui précède le test de leur accessibilité.

## Où les identifiants fuitent

### IDs prévisibles dans les URLs et les paramètres

- **Entiers séquentiels** : `/user/1021`, `?order=5567`. Les incrémenter énumère toute la population et révèle les décomptes et le taux de croissance. Une inscription à deux instants encadre le nombre de comptes existant entre eux.
- **IDs faiblement aléatoires** : IDs numériques courts, valeurs dérivées de timestamp, ou `base64(email)` / `md5(id)` qui s'inversent ou se régénèrent. Les GUID ne sont pas automatiquement sûrs : un UUID version 1 encode un timestamp et une séquence d'horloge plus une valeur de nœud de 48 bits, et lorsque ce nœud est une véritable adresse MAC (plutôt que le nœud aléatoire que la spécification permet aussi), les IDs consécutifs deviennent prévisibles. Ne supposez aucune des deux propriétés ; capturez plusieurs UUID et vérifiez le nibble de version et si les champs de nœud et de timestamp avancent réellement avant de compter sur la prévisibilité.

### Réponses d'API trop bavardes

Les API renvoient fréquemment plus que ce que l'interface affiche. Un endpoint de profil qui montre un nom d'affichage dans la page peut renvoyer l'e-mail, le téléphone, le `user_id` interne, le rôle et le tenant dans le JSON. Inspectez les réponses brutes, pas la page rendue :

- Endpoints de liste et de recherche qui renvoient des enregistrements complets pour chaque ligne.
- Objets qui embarquent des entités liées (une commande portant l'enregistrement client complet).
- GraphQL, où une seule requête peut demander des champs que l'interface ne demande jamais, et où l'**introspection** cartographie chaque type et champ disponible.

### Exports et documents

Les exports CSV, XLSX et PDF sont générés côté serveur et incluent souvent des colonnes que la vue web cache : IDs internes, adresses e-mail, indicateurs de statut, ou lignes soft-supprimées. Les métadonnées dans les PDF et images générés peuvent porter des noms d'utilisateur et des chemins. Demandez chaque export que l'application propose et comparez ses champs à ceux de l'interface.

### Endpoints publics et pré-authentification

- **Pages de profil** à `/u/<name>` confirment l'existence et exposent souvent un ID numérique dans le HTML ou une URL d'avatar liée.
- **Autocomplétion et typeahead** (`/api/users?q=jo`) renvoient les utilisateurs correspondants à tout appelant, une source d'énumération en masse.
- **Messages d'erreur et traces de pile** qui renvoient en écho un ID ou un e-mail.
- **Bundles côté client** : les applications JavaScript et mobiles embarquent des routes d'API, des noms de champs et parfois des identifiants de test ; lisez le bundle.

## Exploitation

Collectez, puis énumérez. Récupérez les IDs depuis un endpoint d'autocomplétion et étendez une plage séquentielle :

```bash
# pull user records across a sequential ID range, keep the ones that resolve
seq 1000 2000 | while read id; do
  curl -s "https://target/api/user/$id" \
    | jq -c 'select(.email) | {id, email, role}' 2>/dev/null
done
```

L'ensemble résultant (id, email, role) est directement utilisable : les e-mails alimentent l'[énumération de comptes](account-enumeration.md) et le [credential stuffing](rate-limits-and-automation.md), et les IDs deviennent des cas de test pour les vérifications de contrôle d'accès sur chaque type d'objet qui s'en sert comme clé. Le bavardage excessif est en soi la trouvaille lorsqu'une réponse ou un export divulgue des données que l'appelant ne devrait pas voir ; l'énumération transforme une fuite unique en une fuite de masse.

## Outils

- **Burp Suite** : Intruder pour parcourir les plages d'IDs ; comparer les champs de réponse ; le sitemap pour repérer les routes portant des IDs.
- **jq** / scripts : extraire et comparer les champs des réponses JSON et des exports.
- **Outillage GraphQL** (requêtes d'introspection, GraphQL Voyager) pour cartographier les champs disponibles.

## Références

- OWASP API Security Top 10 : [API1 Broken Object Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/) et [API3 Broken Object Property Level Authorization / excessive data exposure](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/)
- OWASP WSTG : [Testing for Account Enumeration (WSTG-IDNT-04)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account)
