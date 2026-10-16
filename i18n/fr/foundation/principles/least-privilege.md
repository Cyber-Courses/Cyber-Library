---
title: "Moindre privilège : n'accorder que les accès nécessaires"
description: "Le principe de sécurité selon lequel chaque utilisateur, processus et système ne doit détenir que les permissions requises pour sa tâche, et seulement pendant le temps où il en a besoin."
keywords:
  - moindre privilège
  - principe du moindre privilège
  - contrôle d'accès
  - gestion des privilèges
  - accès juste-à-temps
  - principes de sécurité
---

# Moindre privilège

Le **principe du moindre privilège** énonce que chaque utilisateur, programme et système doit fonctionner avec le plus petit ensemble de permissions dont il a besoin pour accomplir sa tâche, et rien de plus. La logique est simple : un accès qui n'existe pas ne peut pas être détourné, que ce soit par un employé négligent, un compte compromis ou un morceau de code malveillant s'exécutant sous l'identité de quelqu'un.

## La règle en une ligne de Saltzer et Schroeder

Le principe a été formulé en 1975 par Jerome Saltzer et Michael Schroeder comme l'un des huit principes de conception pour protéger l'information dans les systèmes informatiques. Leur formulation est brève : "Chaque programme et chaque utilisateur du système doit fonctionner en utilisant le plus petit ensemble de privilèges nécessaires pour accomplir la tâche."

L'article donne trois raisons qui restent valables aujourd'hui :

- **Limitation des dégâts.** La règle limite "avant tout" les dommages qui peuvent découler d'un accident ou d'une erreur.
- **Moins d'interactions à auditer.** Si seuls quelques programmes détiennent un privilège, alors seuls ces quelques-uns doivent être examinés lorsque ce privilège est détourné.
- **Une carte pour les cloisonnements.** Les auteurs notent que le moindre privilège indique où placer les barrières internes, et ils citent la règle militaire du "besoin d'en connaître" comme exemple de la même idée.

Le même article anticipe aussi une habitude moderne : une même personne peut vouloir agir sous un second principal, moins puissant, lorsqu'elle effectue un travail sans rapport, afin qu'une erreur ne puisse pas toucher l'information précieuse à laquelle son identité principale peut accéder.

## Qui, quoi et pour combien de temps

Appliquer le principe signifie restreindre l'accès selon trois dimensions distinctes. La plupart des techniques concrètes resserrent l'une d'entre elles :

| Dimension | Question à se poser | Technique typique |
|---|---|---|
| Qui | Cette identité a-t-elle réellement besoin d'un accès, et est-elle liée à un propriétaire responsable unique ? | Accès basé sur les rôles, afin que les permissions suivent les fonctions professionnelles et soient perdues lorsque le rôle change ; aucun compte générique partagé |
| Quoi | Quelles ressources et quelles actions, exactement ? | Identifiants à portée limitée pour les applications, les scripts et les charges de travail cloud, restreints aux ressources et opérations spécifiques qu'ils utilisent |
| Quand | L'accès est-il nécessaire maintenant, ou de façon permanente ? | Élévation juste-à-temps et limitée dans le temps, accordée sur demande pour une fenêtre définie et expirant automatiquement |

Une quatrième technique recoupe les trois : les **comptes privilégiés séparés**. Les administrateurs effectuent leur travail quotidien, comme la messagerie et la navigation, depuis un compte standard, et basculent vers un compte distinct et plus puissant uniquement pour les tâches administratives. C'est l'idée du "second principal" de 1975 dans sa forme quotidienne.

## Suivre un seul identifiant volé

Prenons l'exemple d'un attaquant qui obtient par hameçonnage les identifiants d'un développeur dont le compte est utilisé par un pipeline de déploiement.

1. **Version large.** Le compte détient un rôle d'administrateur sur l'ensemble de l'environnement cloud. L'attaquant peut lire toutes les bases de données, créer de nouveaux utilisateurs pour la persistance et désactiver la journalisation. Un seul courriel d'hameçonnage devient une compromission totale.
2. **Version restreinte.** Le pipeline utilise sa propre identité de service qui ne peut que pousser des images vers un seul dépôt et mettre à jour une seule application. Le compte personnel du développeur dispose d'un accès en lecture seule à la production, avec un accès en écriture disponible juste-à-temps après approbation.
3. **Ce à quoi l'attaquant est désormais confronté.** L'identifiant volé n'atteint qu'une petite portion de l'environnement. Pour aller plus loin, l'attaquant doit trouver et exploiter des faiblesses supplémentaires, ce qui prend du temps et génère du bruit que les défenseurs peuvent détecter.
4. **Ce que voient les enquêteurs.** Parce que chaque identité a un objectif clair et minimal, une action en dehors de cet objectif (comme un compte en lecture seule tentant une écriture) ressort dans les journaux et est facile à attribuer.

Le même resserrement protège contre les erreurs honnêtes. De nombreuses pannes et pertes de données proviennent de quelqu'un qui supprime la mauvaise ressource ou modifie un paramètre de production avec un accès dont il n'avait pas besoin pour la tâche en cours.

## Comment les privilèges reviennent insidieusement

Le moindre privilège n'est pas une configuration ponctuelle. L'accès a tendance à croître à moins que quelque chose ne le réduise activement :

- **Dérive des privilèges :** les personnes changent d'équipe ou de projet et conservent l'accès qu'elles avaient auparavant, accumulant lentement bien plus que ce que n'importe quel rôle unique exige.
- **Octrois de confort :** des rôles larges comme "administrateur" ou des politiques cloud avec caractères génériques sont distribués parce que définir des permissions précises demande des efforts.
- **Identifiants permanents et partagés :** des identifiants à privilèges élevés qui n'expirent jamais, et des comptes génériques utilisés par plusieurs personnes, sapent à la fois l'accès minimal et la traçabilité.
- **Identités non humaines oubliées :** les comptes de service, les clés d'API et les jetons d'automatisation sont souvent surprivilégiés et rarement revus, alors même qu'ils peuvent être plus nombreux que les utilisateurs humains.

Le contrepoids est la **revue d'accès régulière** : les permissions sont périodiquement comparées à l'usage et au besoin réels, et les droits inutilisés ou excessifs sont supprimés. NIST SP 800-53 intègre cela dans sa famille de contrôles du moindre privilège (AC-6), qui comprend la revue des privilèges attribués pour confirmer qu'ils sont toujours nécessaires.

## Où il rejoint les autres principes

Le moindre privilège fonctionne rarement seul. L'autorisation, abordée dans [authentification et autorisation](authentication-and-authorization.md), est le point où il est effectivement appliqué, requête par requête. [Le zero trust](zero-trust.md) vérifie chaque requête et se marie naturellement avec le fait de n'accorder à chacune que l'accès minimal dont elle a besoin, ce qui explique pourquoi l'accès juste-à-temps apparaît dans les modèles de maturité zero trust. Au sein de la [défense en profondeur](defense-in-depth.md), des permissions strictes constituent la couche qui empêche un seul compte compromis d'exposer l'ensemble du système. Enfin, la [minimisation](minimization.md) applique le même raisonnement aux données et à la surface d'attaque : garder moins, exposer moins, et il y a moins à perdre.

## Références

- [Saltzer and Schroeder, The Protection of Information in Computer Systems (1975)](https://www.cs.virginia.edu/~evans/cs551/saltzer/)
- [NIST Glossary: Least Privilege](https://csrc.nist.gov/glossary/term/least_privilege)
- [NIST SP 800-53 Rev. 5, Security and Privacy Controls (AC-6 Least Privilege)](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [CISA: Zero Trust Maturity Model](https://www.cisa.gov/zero-trust-maturity-model)
- [OWASP Top 10 Proactive Controls, C1: Implement Access Control](https://top10proactive.owasp.org/the-top-10/c1-accesscontrol/)
- [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
