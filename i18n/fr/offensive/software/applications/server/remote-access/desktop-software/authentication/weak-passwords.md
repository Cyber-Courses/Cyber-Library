---
title: "Mots de passe faibles : deviner les mots de passe de session et sans surveillance du bureau à distance"
description: "Les outils de bureau à distance protègent les connexions par un mot de passe de session ou sans surveillance fréquemment faible : mots de passe de session auto-générés courts, mots de passe sans surveillance faibles choisis par l'utilisateur, et identifiants réutilisés. Ils sont forcés par force brute ou rejoués contre l'ID, et une limitation de débit historiquement faible sur certains produits rendait pratique la devinette en ligne du petit espace de mots de passe."
keywords:
  - weak password
  - session password
  - brute force
  - credential reuse
  - teamviewer
---

# Mots de passe faibles

Le mot de passe est ce qui garde réellement une connexion de bureau à distance, et il est souvent faible. Les mots de passe de session auto-générés sont courts (quelques caractères/chiffres) pour pouvoir être lus à voix haute, ce qui réduit l'espace ; les mots de passe sans surveillance sont choisis par l'utilisateur et fréquemment faibles ou réutilisés ; et tous deux sont sujets à la réutilisation d'identifiants issus de fuites et de logs d'infostealers. Un attaquant force le mot de passe par force brute contre un ID connu ou en rejoue un fuité. Historiquement, une limitation de débit ou un verrouillage insuffisants sur certains produits rendaient pratique la devinette en ligne du petit espace de mots de passe de session à grande échelle, ce qui a alimenté des campagnes massives de prise de contrôle de comptes ; les fournisseurs ont depuis ajouté de la limitation, donc la faisabilité actuelle dépend du produit et de la version.

```bash
# brute force / replay passwords against a known device ID
#   short auto-generated session passwords have a small space (version-dependent)
#   unattended passwords are guessed with common/reused lists
# replay leaked ID:password pairs from breaches/stealer logs for direct connects
```

## Notes d'exploitation

- Les mots de passe de session sont courts par conception (destinés à être prononcés), donc leur espace est petit là où la limitation de débit est faible ; les mots de passe sans surveillance sont la cible plus durable et tombent face aux listes communes/réutilisées.
- La réutilisation d'identifiants est la voie moderne fiable : des paires ID et mot de passe présentes dans les données de fuites et d'infostealers se connectent directement sans devinette, à l'image du credential stuffing VPN/RDP.
- La limitation de débit et le verrouillage des fournisseurs émoussent désormais la pure force brute en ligne sur les versions actuelles ; la praticabilité dépend du produit et de ses protections, donc établissez l'empreinte de la [version](../enumeration/version-detection.md).
- Un mot de passe deviné/rejoué plus l'ID équivaut à un contrôle interactif ; un mot de passe sans surveillance est un accès permanent, voir [accès sans surveillance](unattended-access.md).

## Références

- [TeamViewer security (password handling)](https://www.teamviewer.com/en/trust-center/security/)
- [HackTricks](https://book.hacktricks.xyz/)
