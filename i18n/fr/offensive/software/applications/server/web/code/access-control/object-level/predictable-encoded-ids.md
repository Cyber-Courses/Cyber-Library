---
title: "ID encodés prévisibles : encodages réversibles de clés de base de données dans les URL et les jetons"
description: Identifiants obfusqués (Base64, encodages personnalisés) utilisés à la place d'une autorisation, qui ne font que ralentir les tests manuels au lieu de remplacer les décisions d'accès.
keywords:
  - id encodé
  - obfuscation
  - IDOR
---

# ID encodés prévisibles

## Contexte

Encoder `user_id=7` en Base64 ou selon un schéma personnalisé cache la valeur aux regards occasionnels mais n'est pas un contrôle d'accès. Décoder dans un script et rejouer le même test BOLA comme un identifiant brut est le même constat avec une étape de plus.

## Théorie

Le double encodage, l'encodage URL et les schémas courts à largeur fixe réduisent l'espace de valeurs à deviner en ligne ; ils n'ajoutent pas de contrôle côté serveur. La génération uniquement côté client de jetons « opaques » reste devinable si l'algorithme se trouve dans le binaire mobile.

## Pratique

### Décoder et rejouer en labo

- Copiez un identifiant depuis le trafic, décodez-le en Base64 ou inversez le schéma visible, modifiez l'entier ou la chaîne à l'intérieur, ré-encodez, et substituez-le dans le même appel API. Si l'accès suit la cible modifiée, le serveur n'a jamais imposé la propriété, seulement l'obscurité.

## Outils

- **CyberChef** (en labo)
- **Burp Suite**
- **Python** en une ligne dans un shell de test
