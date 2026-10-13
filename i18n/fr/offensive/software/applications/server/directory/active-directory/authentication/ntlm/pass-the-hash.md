---
order: 4
title: "Pass-the-hash : s'authentifier avec le seul hash NT"
description: "S'authentifier aux services Windows directement avec un hash NT volé au lieu d'un mot de passe, et exécuter à distance via SMB, WMI et WinRM, parce que NTLM dérive sa réponse du hash et n'a jamais besoin du texte en clair."
keywords:
  - pass the hash
  - hash NT
  - overpass the hash
  - mouvement latéral
  - NTLM
---

# Pass-the-hash

La réponse NTLM est calculée à partir du hash NT, pas du mot de passe, donc le **hash est l'identifiant**. Si vous récupérez le hash NT d'un compte ([LSASS](../credentials/lsass-dumping.md), [SAM](../credentials/sam-and-lsa-secrets.md), ou [NTDS/DCSync](../credentials/ntds-and-dcsync.md)), vous pouvez vous authentifier en tant que ce compte partout où NTLM est accepté, sans jamais le cracker. C'est la principale technique de mouvement latéral sous NTLM.

**Lignée.** Le défaut est aussi vieux que le protocole : la réponse NTLM est dérivée directement du hash NT, donc le hash lui-même est un identifiant suffisant, et le hachage faible basé sur DES de LanMan rendait les anciens secrets Windows bon marché à cracker par-dessus le marché. La réutilisation de hash est devenue pratique à la fin des années 1990 et une boîte à outils standard au cours des années 2000. Le vol d'identifiants en mémoire l'a industrialisée, et le mode Restricted Admin, Protected Users et Credential Guard sont les réponses ultérieures qui poussent les opérateurs d'aujourd'hui vers l'overpass-the-hash AES et les pivots Kerberos là où le hash NT brut est refusé.

## Exécuter avec un hash

```bash
# Impacket: remote exec authenticating with the NT hash
psexec.py -hashes :<nthash> EXAMPLE/admin@<host>      # SMB service, SYSTEM
wmiexec.py -hashes :<nthash> EXAMPLE/admin@<host>     # WMI, quieter
smbexec.py -hashes :<nthash> EXAMPLE/admin@<host>

# NetExec: spray a hash across many hosts to map local-admin access
nxc smb <subnet> -u admin -H <nthash> --local-auth
nxc smb <subnet> -u admin -H <nthash>                 # domain account

# Evil-WinRM over WinRM
evil-winrm -i <host> -u admin -H <nthash>
```

Le balayage `-H` / `-hashes` sur un sous-réseau est la manière de trouver **où** un hash récupéré accorde l'admin local, transformant un seul hash extrait en une carte des hôtes atteignables.

## Comptes locaux vs de domaine

- Le hash d'un compte **local** (depuis SAM) fonctionne sur chaque machine partageant ce mot de passe, le problème classique de réutilisation de l'admin local ; LAPS casse cela en randomisant par hôte.
- Le hash d'un compte **de domaine** fonctionne partout où le compte a des droits, donc un hash de domain admin extrait est un accès immédiat à l'échelle du domaine.

## Overpass-the-hash (du hash vers Kerberos)

Le pass-the-hash NTLM peut être bruyant ou bloqué là où seul Kerberos est accepté. **L'overpass-the-hash** utilise le hash NT (ou la clé AES) pour demander un TGT Kerberos, convertissant le hash en un ticket et vous laissant vous déplacer avec Kerberos à la place :

```bash
getTGT.py -hashes :<nthash> EXAMPLE/admin        # request a TGT from the hash
# then export KRB5CCNAME and use -k with the Impacket exec tools
```

Utiliser la **clé AES** au lieu du hash RC4/NT pour cela est plus discret là où RC4 est surveillé ou désactivé. Cela fait le pont vers les techniques Kerberos traitées dans la section Kerberos.

## Notes d'exploitation

- Le pass-the-hash ne nécessite aucun cracking, donc un mot de passe robuste et incrackable n'offre aucune défense une fois son hash récupéré ; seule la rotation du mot de passe (ou un compte différent) invalide le hash.
- L'Administrateur RID-500 intégré est la cible locale à plus forte valeur car son hash SAM est présent sur chaque hôte et souvent réutilisé.
- Là où seul le Kerberos AES est accepté, utilisez le **pass-the-key** (la clé AES) plutôt que le hash NT ; récupérez les clés AES depuis LSASS ou un dump NTDS `-just-dc`.

## Outils

- **Impacket** (`psexec.py`, `wmiexec.py`, `smbexec.py`, `getTGT.py`) : exécution basée sur le hash et overpass-the-hash.
- **NetExec (nxc)** : spraying de hash et cartographie d'accès sur les hôtes.
- **Evil-WinRM / Mimikatz `sekurlsa::pth`** : exécution WinRM et injection de hash sur l'hôte.

## Références

- [The Hacker Recipes: pass the hash](https://www.thehacker.recipes/ad/movement/ntlm/pth)
- [Impacket (fortra): psexec/wmiexec/smbexec and -hashes](https://github.com/fortra/impacket)
- [NetExec: authenticating with -H across protocols](https://www.netexec.wiki/)
- [Microsoft (MS-NLMP): NTLM authentication protocol](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/b38c36ed-2804-4868-a9ff-8dd3182128e4)
