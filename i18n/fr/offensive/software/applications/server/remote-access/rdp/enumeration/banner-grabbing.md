---
title: "Capture de bannière : divulgation de la version et de l'identité RDP"
description: "RDP n'envoie pas de bannière en clair, mais sa poignée de main de connexion divulgue l'identité de l'hôte : la négociation NTLM pendant la mise en place de la sécurité révèle le nom d'hôte NetBIOS et DNS, le domaine et le numéro de build du système d'exploitation. Ce numéro de build date le système et l'associe aux vulnérabilités RDP pré-authentification auxquelles il peut être exposé."
keywords:
  - bannière rdp
  - rdp-ntlm-info
  - build du système d'exploitation
  - nom d'hôte domaine
  - détection de version
---

# Capture de bannière

RDP n'a pas de bannière en clair comme SSH, mais sa poignée de main est instructive. Pendant la mise en place de la connexion, le serveur négocie la sécurité, et lorsque le fournisseur NTLM est impliqué, il renvoie un défi NTLM dont les champs d'informations cibles divulguent les noms NetBIOS et DNS de la machine, le domaine (ou le groupe de travail), le domaine et la forêt DNS, ainsi que la version et le numéro de build du système d'exploitation. Ce numéro de build est le fait clé : il date la version de Windows et l'époque de correctif, ce qui détermine si l'hôte entre dans la plage des bugs RDP pré-authentification vermiformes et quel comportement d'authentification attendre.

```bash
# extract host identity and build from the NTLM handshake
nmap -p3389 --script rdp-ntlm-info <target>
#  => Target_Name, NetBIOS_Domain_Name, DNS_Computer_Name, Product_Version (build)
# confirm the port is RDP and get service/version
nmap -p3389 -sV <target>
```

## Notes d'exploitation

- `rdp-ntlm-info` est l'équivalent RDP d'une bannière : le numéro de build `Product_Version` s'associe directement à la version du système d'exploitation et à la fenêtre de correctif, vous indiquant si [BlueKeep](../pre-authentication-flaws/bluekeep.md) ou [DejaBlue](../pre-authentication-flaws/dejablue.md) pourrait s'appliquer.
- Le nom d'hôte et le domaine divulgués alimentent l'énumération Active Directory et le password spraying ciblé (formats domaine\utilisateur), et confirment si l'hôte est joint à un domaine.
- Cela fonctionne même lorsque NLA est exigée, car l'échange NTLM a lieu pendant la négociation de la sécurité avant une session de bureau ; c'est de la reconnaissance non authentifiée.
- Associez la build aux [réglages de sécurité](security-settings.md) pour compléter la posture avant de choisir une attaque.

## Références

- [nmap rdp-ntlm-info](https://nmap.org/nsedoc/scripts/rdp-ntlm-info.html)
- [HackTricks: RDP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
