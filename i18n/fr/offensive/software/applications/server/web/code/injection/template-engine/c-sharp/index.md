---
title: "Injection de template côté serveur en C#"
description: "SSTI dans les moteurs de template .NET : Razor (RazorEngine et vues compilées à l'exécution) compilant du C# embarqué pour atteindre System.Diagnostics.Process."
keywords:
  - SSTI C#
  - injection de template .NET
  - Razor
  - RazorEngine
---

# C#

L'injection de template .NET est centrée sur Razor, le moteur de vue derrière ASP.NET et les bibliothèques autonomes RazorEngine / RazorLight. Razor compile ses templates en C#, de sorte que lorsqu'une entrée non fiable est compilée comme template Razor, l'injection exécute du C# arbitraire et atteint `System.Diagnostics.Process.Start`, donnant une exécution de commande directe sans sandbox.

Razor utilise `@` pour le code : `@(7*7)` affiche `49`, et les blocs `@{ ... }` exécutent des instructions.

## Moteurs

- **[Razor](razor.md)** : les expressions `@` et les blocs de code compilant en C#, le sink `RunCompile` de RazorEngine/RazorLight, et `Process.Start` pour la RCE.

## Références

- Documentation Microsoft Razor ; projets RazorEngine / RazorLight
- PortSwigger Web Security Academy: Server-side template injection
