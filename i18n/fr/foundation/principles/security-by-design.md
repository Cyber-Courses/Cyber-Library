---
title: "Sécurité dès la conception : intégrer la protection dès le départ"
description: "Le principe selon lequel la sécurité doit être prévue dans l'architecture d'un système dès le début plutôt qu'ajoutée une fois le système construit."
keywords:
  - sécurité dès la conception
  - sécurisé par conception
  - modélisation des menaces
  - cycle de développement sécurisé
  - architecture de sécurité
  - sécurité par défaut
---

# Sécurité dès la conception

**La sécurité dès la conception** consiste à traiter la protection comme une exigence fondamentale d'un système dès ses toutes premières étapes de planification, plutôt que comme une fonctionnalité greffée une fois que le système fonctionne déjà. Lorsque la sécurité est envisagée aux côtés des fonctionnalités, des performances et du coût, l'architecture qui en résulte tend à être plus simple, moins coûteuse à défendre et bien moins fragile qu'une architecture corrigée après sa mise en production. L'idée est étroitement liée à l'objectif voisin d'être sécurisé par défaut, selon lequel un système est livré dans sa configuration la plus sûre et protège ses utilisateurs sans effort supplémentaire de leur part.

## Les principes de conception de 1975 à l'origine de l'idée

La notion est plus ancienne que le slogan moderne. En 1975, Jerome Saltzer et Michael Schroeder ont publié "The Protection of Information in Computer Systems", qui énonçait des principes de conception pour les mécanismes de protection. Plusieurs d'entre eux se lisent aujourd'hui comme le cœur de la sécurité dès la conception :

| Principe de Saltzer et Schroeder | Ce qu'il exige du concepteur |
| --- | --- |
| Économie de mécanisme | Garder une conception petite et simple pour qu'elle puisse réellement être examinée. |
| Valeurs par défaut sûres | Fonder l'accès sur une autorisation explicite ; la réponse par défaut est "non". |
| Médiation complète | Vérifier chaque accès à chaque objet, et pas seulement le premier. |
| Conception ouverte | Ne pas compter sur le secret de la conception elle-même pour assurer la protection. |
| Moindre privilège | N'accorder à chaque programme et à chaque utilisateur que les droits nécessaires à la tâche. |
| Acceptabilité psychologique | Rendre le chemin sécurisé facile, sinon les gens le contourneront. |

Aucun de ces principes ne peut être ajouté proprement à un système déjà terminé. Une base de code tentaculaire ne peut pas être rendue économe après coup, et un modèle d'accès qui autorise par défaut ne peut pas basculer vers un refus par défaut sans casser ce qui a fini par en dépendre.

## Des décisions peu coûteuses au début et chères à la fin

Corriger une faille devient plus coûteux et plus perturbateur à mesure qu'elle est découverte tardivement. Une faiblesse détectée lors d'une revue de conception peut être redessinée sur un tableau blanc ; la même faiblesse découverte en production peut exiger des correctifs d'urgence, des interruptions de service et une réponse à incident. Les choix ci-dessous sont ceux pour lesquels cet écart est le plus grand :

- **Frontières de confiance.** Décider quels composants se font mutuellement confiance, et où les entrées doivent être validées, façonne toutes les interfaces qui suivent.
- **Flux de données.** Les endroits où circulent et sont stockées les données sensibles déterminent ce qui doit être chiffré, journalisé et protégé.
- **Authentification et modèle d'accès.** La manière dont les identités sont établies et les permissions accordées se retrouve dans tout le système et il est pénible de revenir dessus.
- **Surface d'attaque.** Chaque fonctionnalité, chaque port et chaque service livrés deviennent une cible potentielle pour un attaquant et un élément que l'équipe doit maintenir.

De nombreuses vulnérabilités graves proviennent de choix architecturaux de ce type, et non d'erreurs de codage, et aucune quantité de tests ultérieurs ne peut entièrement rattraper des fondations fragiles.

## Une fonctionnalité passée au crible d'une conception sécurisée

Prenons une équipe qui ajoute une fonctionnalité de "partage de document par lien" à une application web. Appliquer la sécurité dès la conception ressemble à peu près à ceci :

1. **Modéliser les menaces avant d'écrire du code.** L'équipe se demande ce qu'elle protège (des documents privés), qui pourrait l'attaquer (quiconque obtient ou devine un lien) et comment (en énumérant les liens, en réutilisant d'anciens liens, en les transférant).
2. **Choisir la valeur par défaut sûre.** Les liens sont créés en mode privé, réservés à des destinataires nommés ; les liens publics sont une option que le propriétaire doit activer délibérément.
3. **Réduire la surface.** Les liens sont longs et aléatoires, expirent après un délai défini et n'accordent qu'un accès en lecture, de sorte que la fonctionnalité n'expose rien dont elle n'a pas besoin.
4. **Échouer en position fermée.** Si le service de permissions est injoignable, le lien renvoie une erreur au lieu de servir le document.
5. **Continuer à vérifier tout au long du cycle de vie.** La conception est revue, le code est analysé et testé au regard du modèle de menaces, et les menaces sont réexaminées lorsque la fonctionnalité évolue.

L'étape 5 est ce que formalise un cycle de développement sécurisé : les tâches de sécurité telles que la revue de conception, l'analyse de code et les tests sont intégrées à chaque phase du développement plutôt que réservées à la fin.

## La sécurité par défaut et le transfert de responsabilité vers les éditeurs

Être sécurisé par défaut est la partie du principe que les utilisateurs perçoivent directement. Les systèmes démarrent dans un état verrouillé, les fonctionnalités à risque restant désactivées jusqu'à ce qu'un utilisateur les active délibérément. À l'inverse, un système livré avec des mots de passe d'exemple, des permissions ouvertes ou des messages d'erreur verbeux reste exposé jusqu'à ce que quelqu'un le durcisse manuellement.

En avril 2023, la Cybersecurity and Infrastructure Security Agency (CISA) des États-Unis et des agences partenaires de plusieurs pays ont publié des recommandations conjointes invitant les fabricants de logiciels à adopter des pratiques de sécurité dès la conception et de sécurité par défaut. Ses trois principes déplacent la responsabilité des clients vers les éditeurs :

- **Assumer la responsabilité des résultats de sécurité des clients**, plutôt que de laisser le durcissement à ceux qui achètent le produit.
- **Adopter une transparence et une responsabilité radicales**, par exemple en communiquant ouvertement sur les vulnérabilités et sur la façon dont elles sont traitées.
- **Montrer l'exemple au plus haut niveau**, afin que la sécurité soit une priorité métier fixée par la direction, et pas seulement une tâche confiée aux ingénieurs.

## Les signes d'une sécurité ajoutée après coup

Un système construit sans sécurité dès la conception présente généralement des symptômes reconnaissables :

- **La sécurité est un point de contrôle final.** Une revue juste avant le lancement constitue le seul contrôle, alors que l'architecture est déjà figée et difficile à modifier.
- **La confiance est supposée, pas conçue.** Les réseaux internes, les logiciels clients ou les entrées provenant d'autres composants sont considérés comme sûrs, ce qui ne laisse aucune défense lorsque cette hypothèse s'effondre.
- **Le durcissement incombe au client.** Le produit ne devient sûr qu'après que quelqu'un a suivi un long guide de configuration.
- **Personne ne peut expliquer le système dans son ensemble.** Fonctionnalités et intégrations se sont accumulées au point que plus personne ne peut raisonner sur son comportement face à une attaque.

La sécurité dès la conception est aussi l'étape où se décident les autres principes. Elle planifie les contrôles en couches sur lesquels repose la [défense en profondeur](defense-in-depth.md), détermine qui obtient quel accès selon le [moindre privilège](least-privilege.md) et fait de la réduction des données et de la surface d'attaque décrite dans la [minimisation](minimization.md) un choix délibéré. Le [Zero Trust](zero-trust.md) est une architecture moderne qui applique le même raisonnement en supprimant la confiance implicite dès le départ.

## Références

- [Saltzer and Schroeder, The Protection of Information in Computer Systems (1975)](https://www.cs.virginia.edu/~evans/cs551/saltzer/)
- [CISA : Secure by Design](https://www.cisa.gov/securebydesign)
- [CISA : Principles and Approaches for Secure by Design Software](https://www.cisa.gov/resources-tools/resources/secure-by-design)
- [NIST SP 800-160 Vol. 1, Engineering Trustworthy Secure Systems](https://csrc.nist.gov/pubs/sp/800/160/v1/r1/final)
- [OWASP : Secure Product Design Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secure_Product_Design_Cheat_Sheet.html)
- [OWASP : Threat Modeling Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html)
