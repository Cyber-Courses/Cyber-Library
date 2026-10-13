---
order: 5
title: "Cracking : transformer hors ligne les hash récupérés en mots de passe"
description: "Casser hors ligne le matériel d'identification Active Directory : hash NTLM, tickets de roasting Kerberos (etype 23 RC4 et etype 17/18 AES), identifiants de domaine mis en cache (DCC2), et NetNTLMv2, avec les modes hashcat et la stratégie."
keywords:
  - hashcat
  - cracking NTLM
  - hash kerberoast
  - DCC2
  - NetNTLMv2
---

# Cracking

Beaucoup des secrets que vous récupérez sont des hash, pas des mots de passe. Le cracking est l'étape hors ligne qui transforme un hash en texte clair que vous pouvez taper, pulvériser ou réutiliser lorsque le hash lui-même n'est pas directement utilisable (tickets de roasting, ouvertures de session en cache, NetNTLM capturé). Il se fait sur votre propre matériel, il est donc indétectable pour la cible et borné uniquement par la robustesse du mot de passe et vos dictionnaires.

## Ce qui se casse, et à quelle vitesse

Le format du hash détermine la faisabilité. Du plus rapide au plus lent :

- **NTLM** (`hashcat -m 1000`) : sans sel et rapide ; les mots de passe faibles tombent vite. Mais un hash NTLM robuste est généralement mieux *utilisé* (pass-the-hash) que cassé.
- **TGS Kerberoast, RC4 (etype 23)** (`-m 13100`) : rapide, la raison pour laquelle le Kerberoasting est si efficace contre les comptes de service.
- **TGS Kerberoast, AES (etype 17/18)** (`-m 19600 / 19700`) : beaucoup plus lent ; un domaine qui force l'AES sur les comptes de service émousse le roasting.
- **AS-REP roast** (`-m 18200`) : vitesse similaire au Kerberoast RC4.
- **NetNTLMv2** (`-m 5600`) : capturé par [poisoning/relais](../ntlm/net-ntlm-capture-and-poisoning.md) ; cassable mais plus lent que NTLM.
- **DCC2 / MSCache2** (`-m 2100`) : ouvertures de session de domaine en cache, délibérément lentes et salées, seuls les mots de passe faibles sont donc réalistes.

```bash
hashcat -m 13100 kerberoast.hash wordlist.txt -r rules/best64.rule     # Kerberoast RC4
hashcat -m 18200 asrep.hash wordlist.txt                                # AS-REP roast
hashcat -m 5600 netntlmv2.hash wordlist.txt                             # captured NetNTLMv2
```

## Stratégie

- **Les dictionnaires plus des règles** battent la force brute pour les mots de passe humains ; de grandes listes issues du monde réel (rockyou et compilations de fuites) avec un jeu de règles récupèrent la majeure partie des mots de passe faibles.
- **Ciblez l'organisation** : nom de l'entreprise, saison-année, et motifs propres à la langue locale (mutés avec des règles) attrapent les mots de passe conformes à la politique mais prévisibles que les listes génériques manquent.
- **Concentrez l'effort là où le hash est rapide et le compte privilégié** : un Kerberoast RC4 d'un compte de service administrateur de domaine est le crack de plus grande valeur ; ne gaspillez pas de cycles sur des hash AES ou DCC2 à moins que le mot de passe ne semble faible.

## Notes d'exploitation

- Le cracking est illimité et silencieux, un roast ou un hash capturé ne coûte donc rien au défenseur à émettre mais peut être travaillé indéfiniment hors ligne.
- Un mot de passe de compte de service ou d'administrateur cassé est réutilisable partout où ce compte est valide et alimente directement le [password spraying](password-spraying.md) de comptes apparentés.
- Si l'AES est appliqué, passez du cracking à l'*utilisation* du matériel (pass-the-hash, pass-the-key, réutilisation de ticket) plutôt que de lutter contre le hash lent.

## Outils

- **hashcat** : cracking GPU sur tous les modes ci-dessus.
- **John the Ripper** : alternative, avec les équivalents `--format` et une bonne prise en charge des règles.
- **hcxtools / name-that-hash** : identifier les formats de hash inconnus avant le cracking.

## Références

- [hashcat: example hashes and modes](https://hashcat.net/wiki/doku.php?id=example_hashes)
- [hashcat: rule-based attack](https://hashcat.net/wiki/doku.php?id=rule_based_attack)
- [John the Ripper (jumbo)](https://github.com/openwall/john)
- [Name-That-Hash (HashPals): identify hash types](https://github.com/HashPals/Name-That-Hash)
