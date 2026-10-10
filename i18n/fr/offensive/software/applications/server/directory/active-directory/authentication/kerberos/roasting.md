---
title: "Roasting : extraire du matériel cassable depuis Kerberos"
order: 1
description: "Obtenir du matériel Kerberos cassable hors ligne sans toucher à un hôte : Kerberoasting des comptes de service, AS-REP roasting des comptes sans pré-authentification, et timeroasting via la synchronisation temporelle non authentifiée."
keywords:
  - kerberoasting
  - AS-REP roasting
  - timeroasting
  - service accounts
  - hashcat
---

# Roasting

Le roasting abuse du fait que certaines parties de l'échange Kerberos sont chiffrées avec la clé à long terme d'un compte. Tout utilisateur du domaine peut *demander* ces blobs chiffrés au KDC, puis les casser hors ligne pour récupérer le mot de passe du compte. Aucun code ne s'exécute sur une cible et aucun droit d'administration n'est nécessaire, ce qui fait du roasting l'une des premières choses à tenter avec un seul identifiant de domaine, et dans un cas avec aucun du tout.

**Filiation.** Le Kerberoasting a été publié en 2014 et misait sur le cassage rapide hors ligne des tickets de service RC4. Des types de chiffrement AES plus robustes, les comptes de service gérés par groupe (gMSA) et les politiques de mot de passe affinées ont relevé la barre, et l'AS-REP roasting et le timeroasting sont les dérivés à moindres prérequis qui ont suivi, ne nécessitant qu'un identifiant ou aucun.

## Kerberoasting

Lorsque vous demandez un ticket de service (TGS) pour un compte doté d'un **Service Principal Name**, une partie du ticket est chiffrée avec la clé dérivée du mot de passe de ce compte de service. Demandez des tickets pour chaque compte porteur d'un SPN et cassez les blobs hors ligne :

```bash
# Impacket: request TGS for all SPN accounts (exclude computers via the query)
GetUserSPNs.py -request -dc-ip <dc> example.local/user:pass -outputfile kerb.hash

# Rubeus on-host
Rubeus.exe kerberoast /outfile:kerb.hash

# NetExec, straight to a hash file over LDAP
nxc ldap <dc> -u user -p pass --kerberoasting kerb.hash

hashcat -m 13100 kerb.hash wordlist.txt -r rules/best64.rule     # RC4 (etype 23)
```

- Ciblez les tickets **RC4 (etype 23)** : `-m 13100` casse bien plus vite qu'AES (`-m 19600/19700`). Demandez RC4 explicitement là où le compte l'autorise.
- **Excluez les comptes machine** : leurs mots de passe sont des chaînes aléatoires de 120 caractères qui ne se casseront jamais, filtrez donc sur `objectCategory=person`.
- Les comptes de service sont démesurément privilégiés et leurs mots de passe démesurément faibles (définis une fois, il y a des années), ce qui explique que le Kerberoasting soit si productif.

## AS-REP roasting

Si un compte a l'option **« Ne pas demander la pré-authentification Kerberos »** activée (`DONT_REQ_PREAUTH` dans l'UAC), le KDC renvoie une AS-REP dont la partie chiffrée est dérivée de la clé du compte, à **quiconque**, sans aucun identifiant requis :

```bash
# Impacket: find and roast accounts without pre-auth (works unauthenticated with a user list)
GetNPUsers.py example.local/ -usersfile users.txt -no-pass -dc-ip <dc>
GetNPUsers.py example.local/user:pass -request          # authenticated: enumerate + roast
nxc ldap <dc> -u user -p pass --asreproast asrep.hash   # NetExec

hashcat -m 18200 asrep.hash wordlist.txt                # AS-REP (etype 23)
```

C'est le seul roast qui peut s'exécuter depuis une position totalement non authentifiée, à condition de disposer d'une liste de noms d'utilisateurs candidats.

## Timeroasting

Le timeroasting abuse du NTP non authentifié : le service SNTP d'un contrôleur de domaine renvoie un authentificateur calculé sur le **RID et la clé d'un compte machine**, sans aucun identifiant requis. Les collecter fournit du matériel cassable pour les comptes machine :

```bash
# Request SNTP authenticators keyed by RID, producing hashcat-crackable output
timeroast.py <dc> -o timeroast.hash
hashcat -m 31300 timeroast.hash wordlist.txt
```

Les mots de passe des comptes machine sont généralement robustes, de sorte que le timeroasting trouve surtout des mots de passe d'ordinateur non par défaut ou définis manuellement, mais il ne nécessite aucune authentification.

## Notes d'exploitation

- Un compte de service cassé est immédiatement réutilisable : il dispose souvent de droits d'administrateur local sur ses serveurs applicatifs et alimente le [pass-the-ticket](pass-the-ticket.md) et les [silver tickets](forged-tickets.md).
- Demandez des tickets **RC4** partout où c'est possible ; un domaine qui impose des clés AES uniquement sur les comptes de service émousse la vitesse de cassage, même si le ticket est tout de même émis.
- Le Kerberoasting ne nécessite qu'un seul identifiant valide, l'AS-REP roasting peut n'en nécessiter aucun, les deux ont donc leur place dès le tout début d'un engagement, aux côtés de l'[énumération](spn-discovery.md).

## Outils

- **Impacket** (`GetUserSPNs.py`, `GetNPUsers.py`) : Kerberoast et AS-REP roast.
- **Rubeus** (`kerberoast`, `asreproast`) : roasting depuis l'hôte avec contrôle de l'etype.
- **hashcat** : modes 13100 (Kerberoast), 18200 (AS-REP), 31300 (timeroast).

## Références

- [GhostPack Rubeus (kerberoast / asreproast)](https://github.com/GhostPack/Rubeus)
- [ropnop Kerbrute](https://github.com/ropnop/kerbrute)
- [SecuraBV Timeroast (Tom Tervoort)](https://github.com/SecuraBV/Timeroast)
- [hashcat: example hashes and modes](https://hashcat.net/wiki/doku.php?id=example_hashes)
