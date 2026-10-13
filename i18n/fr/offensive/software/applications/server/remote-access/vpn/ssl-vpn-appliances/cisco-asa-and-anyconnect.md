---
order: 5
title: "Cisco ASA et AnyConnect : traversée WebVPN et attaques d'informations d'identification du portail"
description: "Le VPN d'accès distant de Cisco ASA et Firepower a connu une traversée de chemin WebVPN donnant une divulgation de fichiers non authentifiée, et le portail SSL-VPN AnyConnect est une cible persistante de brute force d'informations d'identification et de password spraying. Combinés, la divulgation de fichiers et des informations d'identification valides accordent l'accès au VPN et au réseau interne qu'il protège."
keywords:
  - cisco asa
  - anyconnect
  - webvpn
  - traversée de chemin
  - password spray
---

# Cisco ASA et AnyConnect

Cisco ASA (et Firepower) fournissent un VPN d'accès distant via le SSL-VPN AnyConnect, et deux voies reviennent. L'interface WebVPN a connu une vulnérabilité de traversée de chemin permettant à un attaquant non authentifié de lire des fichiers de l'appareil, divulguant la configuration et d'autres données utiles à un accès ultérieur. Et le portail SSL-VPN AnyConnect est une cible continuelle d'attaques d'informations d'identification : parce qu'il s'authentifie auprès de l'annuaire d'entreprise et est exposé sur Internet, le password spraying et le credential stuffing contre lui constituent une voie d'accès initial courante et efficace, surtout là où la MFA est absente ou appliquée de manière incohérente. Le VPN d'accès distant ASA a aussi connu des campagnes ciblées combinant ces éléments à une configuration faible.

```bash
# fingerprint ASA/AnyConnect WebVPN
curl -skI https://<gateway>/
curl -sk https://<gateway>/+CSCOE+/logon.html | grep -i 'anyconnect\|cisco'
# WebVPN path-traversal class: unauthenticated file read of appliance files
curl -sk --path-as-is 'https://<gateway>/+CSCOU+/../+CSCOE+/<traversal-target>'
# portal credential attack: spray/stuff the AnyConnect logon (directory-backed)
```

## Notes d'exploitation

- La traversée WebVPN est une lecture de fichiers non authentifiée ; visez la configuration de l'appareil et tout secret divulgué pour permettre ou compléter un accès authentifié.
- Le portail AnyConnect est une cible de spraying/stuffing de premier ordre parce qu'il valide des informations d'identification d'entreprise et est exposé sur Internet ; un mot de passe sprayé ou issu d'une fuite sans MFA est un tunnel vers l'intérieur, voir [brute force de mot de passe](../authentication/password-brute-force.md).
- Surveillez les voies d'authentification qui sautent la MFA (certaines group-URLs, configurations héritées) ; les mauvaises configurations du VPN d'accès distant ASA ont permis un accès sans MFA même là où la MFA est nominalement déployée.
- Les deux donnent accès au réseau interne que l'ASA protège ; identifiez la version ASA/Firepower pour la traversée et repérez les group-URLs AnyConnect pour les attaques d'informations d'identification.

## Références

- [Avis de sécurité Cisco](https://sec.cloudapps.cisco.com/security/center/publicationListing.x)
- [CISA: Cisco ASA remote-access exploitation](https://www.cisa.gov/news-events/cybersecurity-advisories)
