---
order: 1
title: "Énumération : détecter WinRM et sa configuration"
description: "WinRM se reconnaît à ses listeners sur 5985 (HTTP) et 5986 (HTTPS), qui répondent à une sonde WS-Management. L'énumération confirme le service, s'il utilise HTTP ou HTTPS, et quelles méthodes d'authentification sont autorisées, et une vérification d'identifiant révèle si un compte donné peut réellement atteindre WinRM et s'il dispose de droits administratifs."
keywords:
  - winrm enumeration
  - 5985
  - 5986
  - wsman
  - auth methods
---

# Énumération

L'énumération WinRM confirme le service et sa posture avant l'attaque. Les listeners sont 5985 (HTTP) et 5986 (HTTPS), et ils répondent à une requête WS-Management (un POST HTTP vers `/wsman`) de façon distinctive, de sorte qu'une sonde identifie WinRM même s'il repose sur HTTP. L'énumération établit aussi quel transport est en usage et quelles méthodes d'authentification le listener autorise (Negotiate/NTLM, Kerberos, Basic, CredSSP). Enfin, une vérification d'identifiant contre WinRM montre si un compte donné est réellement autorisé à se connecter (appartenance à Remote Management Users ou admin) et si cette connexion est privilégiée.

```bash
nmap -p5985,5986 -sV <target>
# WS-Management probe
curl -s -X POST http://<target>:5985/wsman -H 'Content-Type: application/soap+xml' | head
# validate an account and its WinRM access/privilege
nxc winrm <target> -u user -p 'pass'           # (Pwn3d! indicates admin via WinRM)
```

## Notes d'exploitation

- Les ports 5985/5986 ouverts plus une réponse WS-Management confirment WinRM ; un `401` à la sonde confirme quand même le service et révèle les méthodes d'authentification offertes dans les en-têtes `WWW-Authenticate`.
- La vérification d'identifiant est l'énumération décisive : `nxc winrm` indique si le compte peut utiliser WinRM du tout (tout compte valide ne le peut pas) et s'il atterrit en tant qu'admin.
- HTTPS (5986) enveloppe le même protocole dans TLS ; il ne change pas l'attaque, seulement le transport, bien qu'il bloque la capture passive d'un identifiant en authentification Basic.
- Un endpoint WinRM confirmé plus un identifiant ou un hash utilisable mène directement à l'[authentification](authentication.md) et à l'[exécution de commandes](command-execution.md).

## Références

- [HackTricks: WinRM enumeration](https://book.hacktricks.xyz/network-services-pentesting/5985-5986-pentesting-winrm)
- [NetExec winrm](https://www.netexec.wiki/winrm-protocol)
