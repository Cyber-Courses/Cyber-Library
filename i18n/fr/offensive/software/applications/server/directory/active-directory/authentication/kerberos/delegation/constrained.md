---
title: "Délégation contrainte : S4U et transition de protocole"
description: "Abus de la délégation contrainte Kerberos : utilisation de S4U2self et S4U2proxy depuis un compte de délégation compromis pour obtenir des tickets de service en tant que n'importe quel utilisateur, y compris le cas de la transition de protocole qui ne nécessite aucune authentification préalable de l'utilisateur."
keywords:
  - constrained delegation
  - S4U2self
  - S4U2proxy
  - protocol transition
  - msDS-AllowedToDelegateTo
---

# Délégation contrainte

La délégation contrainte permet à un compte de demander des tickets de service pour un utilisateur, mais uniquement vers une **liste fixe de SPN** indiqués dans son attribut `msDS-AllowedToDelegateTo`. Si vous compromettez un tel compte (son mot de passe, son hash ou sa clé AES), vous pouvez usurper l'identité d'utilisateurs arbitraires auprès de ces services listés, et bien souvent au-delà de ce que la liste autorise littéralement.

## L'abus avec transition de protocole

Lorsque le compte est configuré pour la **transition de protocole** (`TRUSTED_TO_AUTH_FOR_DELEGATION`), il peut appeler **S4U2self** pour forger un ticket vers lui-même en tant que *n'importe quel* utilisateur, sans la moindre intervention de cet utilisateur, puis **S4U2proxy** pour le transformer en ticket destiné au service back-end autorisé en tant que cet utilisateur :

```bash
# Impacket: impersonate Administrator to an allowed SPN, using the delegating account's hash
getST.py -spn cifs/target.example.local -impersonate Administrator \
  -hashes :<delegating-acct-hash> example.local/delegating-acct
export KRB5CCNAME=Administrator.ccache
```

Sans transition de protocole, S4U2self fonctionne toujours, mais le ticket obtenu n'est pas transférable (forwardable), si bien que S4U2proxy est censé le refuser. En pratique, le ticket reste souvent utilisable, et des astuces self/U2U peuvent en produire un qui fonctionne.

## Bronze Bit : forcer le flag forwardable

L'attaque **Bronze Bit** supprime les deux contraintes ci-dessus. Lorsque vous détenez la clé du service de délégation, vous pouvez prendre le ticket S4U2self, **basculer vous-même son flag `forwardable`** (le déchiffrer avec la clé du service, positionner le flag, le rechiffrer), puis le soumettre à S4U2proxy. Le KDC l'accepte, de sorte que la délégation réussit **sans transition de protocole**, et surtout cela vous permet aussi d'usurper l'identité de comptes censés être non délégables : les membres de **Protected Users** et les comptes marqués **« sensibles et ne pouvant pas être délégués »**. Microsoft a restreint ce comportement avec une mise à jour de novembre 2020 (le KDC valide désormais le flag), si bien que cela s'applique aux DC non corrigés.

```bash
# Impacket getST implements it with -force-forwardable
getST.py -spn cifs/target.example.local -impersonate Administrator -force-forwardable \
  -hashes :<delegating-acct-hash> example.local/delegating-acct
```

## L'astuce de substitution de SPN

S4U2proxy contraint la *classe de service* de manière lâche : le **nom de service** du ticket renvoyé peut être échangé contre un autre SPN (le KDC ne lie pas étroitement le ticket à la chaîne de SPN exacte). Le hic tient à la clé : le ticket est chiffré avec la clé du compte back-end, donc le SPN substitué ne fonctionne que s'il est **enregistré sur ce même compte**. En pratique, les SPN de service propres à un hôte (`cifs/`, `host/`, `http/`, `ldap/` sur un DC, etc.) sont tous adossés au **compte machine**, de sorte qu'une délégation autorisée vers l'un d'eux peut être échangée vers les autres sur cet hôte, transformant une délégation d'apparence limitée en contrôle total de cette machine. En revanche, cela n'atteint **pas** un SPN enregistré sur un compte de service *distinct* et dédié, car la clé de ce compte ne peut pas déchiffrer le ticket.

## Notes d'exploitation

- Usurpez l'identité d'un utilisateur privilégié (Domain Admin) vers un SPN `cifs/` ou `host/` sur la cible pour obtenir un accès aux fichiers ou l'exécution de code en tant que cet utilisateur sur cet hôte.
- L'astuce de substitution de SPN implique que le SPN listé importe moins que le **compte cible** : une délégation vers un SPN de compte machine équivaut de fait à une délégation vers tous les services propres de cette machine, mais pas vers les SPN détenus par des comptes de service distincts sur le même hôte.
- Vous avez besoin du secret du compte de délégation ; récupérez-le par [cassage](../../credentials/cracking.md) (il s'agit souvent d'un compte de service) ou par extraction, puis pilotez S4U avec son hash ou sa clé AES.
- Les comptes marqués « sensibles, ne pouvant pas être délégués » et les membres de Protected Users ne peuvent pas être usurpés de cette manière **dans un comportement normal (corrigé)** ; l'attaque Bronze Bit ci-dessus lève précisément cette restriction sur les DC non corrigés.

## Outils

- **Impacket** (`getST.py` avec `-impersonate`) : S4U2self/S4U2proxy et substitution de SPN.
- **Rubeus** (`s4u`) : abus de la délégation contrainte depuis l'hôte.
- **findDelegation.py** (Impacket) : énumérer les comptes configurés pour la délégation.

## Références

- [Elad Shamir: Wagging the Dog](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)
- [Impacket getST (-impersonate)](https://github.com/fortra/impacket)
- [GhostPack Rubeus (s4u)](https://github.com/GhostPack/Rubeus)
