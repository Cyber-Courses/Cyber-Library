---
title: "Sécurité dès la conception : intégrer la protection dès le départ"
description: "La pratique consistant à planifier la sécurité dans l'architecture d'un système dès les premières décisions de conception, plutôt que de l'ajouter une fois que le système fonctionne déjà."
keywords:
  - sécurité dès la conception
  - sécurisé dès la conception
  - modélisation des menaces
  - sécurité décalée vers l'amont
  - cycle de développement sécurisé
  - sécurisé par défaut
---

# Sécurité dès la conception

La **sécurité dès la conception** est la pratique consistant à traiter la protection comme une exigence qui façonne un système dès ses premières décisions de conception, et non comme une couche ajoutée une fois que le système fonctionne déjà. Lorsque la sécurité est pesée aux côtés des fonctionnalités, des performances et du coût dès le départ, l'architecture qui en résulte est généralement plus simple à défendre et bien moins fragile qu'une architecture corrigée après la publication. L'habitude inverse, livrer d'abord et sécuriser ensuite, tend à laisser des failles coûteuses à combler et faciles à trouver pour un attaquant.

## Décaler vers l'amont : quand les décisions sont prises

L'essentiel de la sécurité d'un système est déterminé bien avant que quiconque n'écrive un code d'exploitation contre lui. Les choix relatifs aux frontières de confiance, aux flux de données, aux paramètres par défaut et à qui peut communiquer avec quoi sont faits lors de la conception, et ils sont difficiles à inverser une fois que le code, les données et les utilisateurs en dépendent. "Décaler vers l'amont" signifie ramener l'activité de sécurité vers l'extrémité la plus précoce et la moins coûteuse du calendrier de développement :

- **Exigences.** Les objectifs de sécurité sont consignés sous forme de critères d'acceptation, et non de souhaits.
- **Conception.** L'architecture est examinée pour détecter les frontières de confiance faibles avant l'implémentation.
- **Construction et test.** Des contrôles automatisés détectent les régressions tant qu'une correction reste un petit changement.

Le but n'est pas de faire le travail de sécurité une seule fois, tôt, puis de s'arrêter. Il s'agit de faire des premières décisions celles qui portent la charge.

## Modéliser les menaces avant la première ligne de code

L'activité centrale au moment de la conception est la modélisation des menaces : un examen structuré de ce que vous construisez, de ce qui peut mal tourner et de ce que vous allez y faire. Une version simple répond à quatre questions, souvent attribuées à l'équipe à l'origine du Security Development Lifecycle de Microsoft :

1. **Que construisons-nous ?** Un schéma des composants, des magasins de données et des frontières de confiance entre eux.
2. **Qu'est-ce qui peut mal tourner ?** L'usurpation, l'altération, la divulgation d'informations et des catégories de défaillance similaires, parcourues frontière par frontière.
3. **Qu'allons-nous y faire ?** Une mesure d'atténuation pour chaque menace qui mérite d'être traitée.
4. **Avons-nous bien travaillé ?** Une revue qui confronte le modèle au système réellement construit.

Parce qu'elle travaille sur des schémas, la modélisation des menaces peut exposer une hypothèse de confiance erronée des semaines avant que cette hypothèse ne devienne du code.

## Le coût croissant d'une correction tardive

Les études sur les défauts logiciels ont retrouvé à maintes reprises la même forme : plus un défaut est détecté tard, plus son élimination coûte cher, parce que davantage de travail a été bâti par-dessus.

| Étape où le défaut est détecté | Ce qu'une correction touche | Coût relatif |
|---|---|---|
| Conception | Un schéma ou une décision | Le plus bas |
| Développement | Du code pas encore livré | Faible |
| Test | Du code plus les suites de tests | Modéré |
| Production | Systèmes en service, données et utilisateurs | Le plus élevé |

Un contrôle d'autorisation manquant est une note d'une ligne dans une revue de conception et une réponse à incident coordonnée une fois en production. La sécurité dès la conception cherche à l'attraper tant qu'elle n'est encore que la note d'une ligne.

## Sécurisé dès la conception n'est pas la même chose que sécurisé par défaut

Les deux idées sont souvent associées mais restent distinctes. La sécurité dès la conception concerne le processus : intégrer la protection de manière délibérée. Le sécurisé par défaut concerne le résultat livré : la configuration du produit telle qu'elle sort de la boîte est sa configuration sûre, de sorte qu'un utilisateur est protégé sans avoir à trouver et à activer des commutateurs de sécurité. Un système peut être conçu avec soin et pourtant être livré grand ouvert, et l'attente moderne, portée par des organismes comme la CISA, est que les fournisseurs livrent les deux.

## Là où elle s'appuie sur les autres principes

La sécurité dès la conception est un cadre que les autres principes viennent remplir. Une conception saine suppose qu'aucun contrôle n'est parfait à lui seul et les superpose, ce qui constitue la [défense en profondeur](defense-in-depth.md). Elle n'accorde à chaque composant que ce dont il a besoin, en appliquant le [moindre privilège](least-privilege.md), et elle ne collecte que les données dont elle a réellement besoin grâce à la [minimisation](minimization.md). Décider de la confiance au moment de la conception plutôt que de l'hériter du réseau est au cœur du [zero trust](zero-trust.md).

## Références

- [Saltzer and Schroeder, The Protection of Information in Computer Systems (1975)](https://www.cs.virginia.edu/~evans/cs551/saltzer/)
- [CISA, Secure by Design](https://www.cisa.gov/securebydesign)
- [Microsoft Security Development Lifecycle (SDL) Practices](https://www.microsoft.com/en-us/securityengineering/sdl/practices)
- [OWASP Developer Guide: Secure Design Principles](https://owasp.org/www-project-developer-guide/)
- [NIST SP 800-160 Vol. 1, Engineering Trustworthy Secure Systems](https://csrc.nist.gov/pubs/sp/800/160/v1/r1/final)
