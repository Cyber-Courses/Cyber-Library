---
title: "Gates de workflow uniquement en session : indicateurs d'étape sans corroboration faisant autorité en base de données"
description: "Étapes conditionnées uniquement par des indicateurs de session qui ne sont pas liés à la ligne de workflow faisant autorité."
keywords:
  - fixation de session
  - workflow
---

# Points de contrôle de session

## Contexte

`session['checkout_step']=3` débloque le formulaire de paiement, mais la ligne de commande brouillon a été supprimée ou annulée. L'utilisateur soumet le paiement sur un panier incohérent, ou le serveur accepte le paiement sans relire le stock.

## Théorie

Chaque étape modifiant l'état devrait valider un jeton stocké côté serveur ou un identifiant de commande qui lie la session à la ligne.

## Pratique

- Ouvrez deux sessions indépendantes pour le même compte (deux navigateurs ou fenêtres de navigation privée, chacune avec son propre cookie de session). Dans la session A, avancez jusqu'à l'étape conditionnée pour que sa session contienne `checkout_step=3` (ou l'indicateur équivalent). Dans la session B, modifiez l'enregistrement faisant autorité à son insu : annulez la commande brouillon, videz le panier, ou faites passer la ligne dans un état qui devrait révoquer l'étape. Soumettez ensuite la requête modifiant l'état depuis la session A, dont le point de contrôle périmé indique toujours que l'étape est débloquée. Si le serveur honore l'indicateur de session au lieu de relire la ligne, l'action incohérente est validée.

## Outils

- Deux navigateurs ou fenêtres de navigation privée (sessions distinctes) sur le même compte en staging

## Références

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
