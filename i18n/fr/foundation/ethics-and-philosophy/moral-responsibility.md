---
title: "Responsabilité morale : le devoir qui accompagne le pouvoir sur les systèmes"
description: "L'obligation que les praticiens de la sécurité ont envers les utilisateurs, les organisations et le public, parce que leur travail leur confère un véritable pouvoir sur des systèmes qui affectent la vie des gens."
keywords:
  - responsabilité morale
  - éthique de la sécurité
  - responsabilité professionnelle
  - devoir de diligence
  - pratique responsable
  - éthique de la technologie
---

# Responsabilité morale

**La responsabilité morale** est le devoir qu'un praticien porte parce que son travail lui confère un pouvoir sur des systèmes dont d'autres personnes dépendent. Quelqu'un qui peut lire une base de données, corriger un serveur ou décider du moment de divulguer une faille fait des choix qui vont bien au-delà de son propre écran. Ces choix touchent des utilisateurs qui n'ont jamais consenti au risque, des organisations qui ont confié un accès au praticien, et un public qui vit avec les conséquences. Plus une personne détient de capacités, plus ce devoir devient lourd.

## Contrôle et connaissance : les deux conditions

Depuis Aristote, les philosophes rattachent la responsabilité à deux conditions : une personne doit avoir un certain **contrôle** sur ce qu'elle fait, et elle doit **savoir**, ou pouvoir savoir, ce qu'elle fait et ce que cela peut provoquer. Le travail de sécurité remplit ces deux conditions de manière particulièrement forte, ce qui explique pourquoi le devoir pèse ici plus lourd que dans la plupart des métiers.

| Condition | Ce qu'elle demande | Pourquoi les praticiens de la sécurité la remplissent |
| --- | --- | --- |
| Contrôle | La personne aurait-elle pu agir autrement ? | Un accès privilégié signifie qu'un seul administrateur peut exposer ou protéger des millions d'enregistrements, et qu'un chercheur décide du sort d'une faiblesse qu'il a découverte. |
| Connaissance | La personne comprenait-elle le résultat probable ? | L'expertise supprime l'excuse de l'ignorance : un professionnel qui comprend le risque est responsable d'une manière qu'un profane ne l'est pas. |

Une caractéristique supplémentaire aiguise ce devoir : **le préjudice est souvent invisible pour celui qui agit.** La personne affectée par un enregistrement divulgué ou une suppression négligente est rarement présente lorsque la décision est prise, si bien que le praticien doit délibérément la garder à l'esprit. Se voir confier les clés d'un système est une forme de confiance, et pas seulement un fait technique.

## Quatre habitudes qui portent le devoir

La responsabilité morale se manifeste rarement par un grand geste. Elle vit dans des décisions de jugement ordinaires, et chacune des habitudes ci-dessous a une façon reconnaissable d'échouer.

1. **Devoir de diligence.** Traiter les données et les accès avec la prudence que le propriétaire attendrait, même lorsque personne ne regarde. L'échec ressemble à la copie d'une base de données de production sur un ordinateur portable "juste pour tester un truc".
2. **Divulgation honnête.** Signaler rapidement les failles, les incidents et les erreurs, y compris les siennes. L'échec ressemble à la correction discrète d'une erreur de configuration que l'on a soi-même causée, sans jamais mentionner la période d'exposition.
3. **Action proportionnée.** N'utiliser que l'accès et la force qu'une tâche exige, et s'arrêter une fois le travail terminé. L'échec ressemble à un testeur d'intrusion qui pivote vers des systèmes hors du périmètre convenu simplement parce qu'il le pouvait. La question de savoir si vous étiez autorisé à agir tout court relève de [l'autorisation et du consentement](authorization-and-consent.md).
4. **Assumer les conséquences.** Assumer le résultat d'une décision plutôt que de se cacher derrière une politique ou un ordre. L'échec ressemble à "je n'ai fait qu'exécuter le script qu'on m'a donné".

Ces habitudes correspondent aux codes professionnels. Le Code d'éthique de l'(ISC)2 s'ouvre sur le principe de "protéger la société, le bien commun, la confiance publique nécessaire et l'infrastructure", et les codes de l'ACM et de l'IEEE demandent de même à leurs membres de placer l'intérêt public au-dessus de la commodité ou de la pression de l'employeur.

## Des mains nombreuses, personne de responsable

Les systèmes modernes sont construits et exploités par des équipes, des fournisseurs et des sous-traitants, et la responsabilité peut se diluer au point que personne ne la ressent. C'est ce qu'on appelle le **problème des mains nombreuses** : lorsque de nombreuses personnes contribuent à un résultat, il devient difficile de dire qui était responsable de quoi. Helen Nissenbaum l'a identifié en 1996 comme l'un des principaux obstacles à la responsabilité en informatique.

Le cas classique est celui du Therac-25, un appareil de radiothérapie dont les défauts logiciels ont contribué à une série de surdosages massifs au milieu des années 1980. Les développeurs, les testeurs, le fabricant et les opérateurs détenaient chacun une part de l'échec, et le choix d'aucune personne ne l'explique à lui seul. En sécurité, le même schéma apparaît lorsqu'une faiblesse connue reste des mois dans un backlog parce que chaque équipe suppose qu'une autre en est propriétaire. La réponse pratique consiste à désigner un responsable pour chaque risque et à traiter "ce n'est pas mon composant" comme une question à résoudre, et non comme une conclusion.

## Quand l'employeur veut le silence : un scénario de divulgation

Prenons une ingénieure qui découvre qu'un service exposé aux clients divulgue les informations de compte d'autres clients. La direction lui demande de garder le silence jusqu'à la prochaine version trimestrielle, dans trois mois.

- **L'attrait de la loyauté.** Elle doit à son employeur un service diligent et compétent, et une annonce publique prématurée pourrait offrir une carte aux attaquants avant qu'un correctif n'existe.
- **L'attrait du public.** Chaque jour de silence laisse les utilisateurs exposés à un risque qu'ils ignorent et contre lequel ils ne peuvent pas se défendre. Ses codes d'éthique placent leur intérêt en premier.
- **Le coût de chaque voie.** Agir sur un risque connu peut en soi causer un préjudice, et attendre aussi. L'inaction est également un choix, et elle porte un poids moral.

Une voie défendable se situe généralement entre le silence et la fuite publique : insister en interne pour obtenir un correctif urgent, documenter le risque et qui l'a accepté, et en cas d'échec, recourir à un canal structuré tel que le processus de divulgation coordonnée géré par la CISA, qui existe précisément pour corriger les failles avant que les détails ne soient rendus publics. La même tension entre protéger les personnes et les exposer apparaît dans [sécurité contre vie privée](security-versus-privacy.md), et la question de savoir quelle part de pouvoir sur autrui un praticien devrait détenir dès le départ est abordée dans [liberté contre contrôle](freedom-versus-control.md).

## Les ordres ne transfèrent pas la responsabilité

Suivre des instructions ne fait pas passer le poids moral sur celui qui les a données. Un ordre légal peut tout de même être la mauvaise chose à faire, et "on me l'a demandé" explique un choix sans l'excuser. Cela compte surtout pour les praticiens dont les compétences sont à double tranchant : l'expertise qui renforce un système peut aussi être retournée contre lui, comme l'explore [la connaissance à double usage](dual-use-knowledge.md), si bien que la personne qui détient cette expertise ne peut pas confier entièrement à quelqu'un d'autre la décision de son usage.

Il n'existe aucune formule qui résolve ces conflits proprement. La responsabilité morale, c'est la volonté de peser honnêtement les prétentions concurrentes et de répondre de son choix.

## Références

- [(ISC)2 Code of Ethics](https://www.isc2.org/ethics)
- [ACM Code of Ethics and Professional Conduct](https://www.acm.org/code-of-ethics)
- [IEEE Code of Ethics](https://www.ieee.org/about/corporate/governance/p7-8.html)
- [Stanford Encyclopedia of Philosophy: Moral Responsibility](https://plato.stanford.edu/entries/moral-responsibility/)
- [Stanford Encyclopedia of Philosophy: Computing and Moral Responsibility](https://plato.stanford.edu/entries/computing-responsibility/)
- [CISA: Coordinated Vulnerability Disclosure Process](https://www.cisa.gov/coordinated-vulnerability-disclosure-process)
