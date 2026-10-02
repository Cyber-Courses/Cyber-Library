---
title: "Exposed config and dotfiles: .env, config files, and metadata left web-readable"
description: "Recovering credentials and secrets from configuration and dotfiles served as static content: .env, framework config, .htaccess, CI and editor dotfiles, and cloud metadata files."
keywords:
  - .env exposure
  - config file disclosure
  - dotfiles
  - secrets
  - htaccess
---

# Exposed config and dotfiles

Configuration files hold the secrets everything else depends on: database credentials, API keys, framework signing keys, and third-party tokens. When they sit in the web root and the server returns them as static text (because the server only executes `.php`, not `.env` or `.yml`), a single request exfiltrates them.

## High-value targets

```
/.env                      # Laravel/Symfony/Node: DB creds, APP_KEY, API keys
/.env.local  /.env.prod    # environment variants
/config.php.txt /config.yml /settings.py /appsettings.json
/wp-config.php.bak         # backup variant is served as source
/.htaccess  /.htpasswd     # rules, and hashed creds
/docker-compose.yml  /Dockerfile   # service creds, build detail
/.aws/credentials  /.npmrc  /.pypirc   # cloud and registry tokens
/.DS_Store                 # macOS directory metadata -> filenames
/package.json  /composer.lock  /yarn.lock   # dependency inventory
```

`.env` is the highest-yield single file: it typically contains the database DSN, the framework `APP_KEY`/`SECRET_KEY` (which forges signed cookies, sessions, and password-reset tokens), mail creds, and cloud keys.

## Finding and reading

```
GET /.env HTTP/1.1
GET /.git/config HTTP/1.1
GET /.DS_Store HTTP/1.1
```

`.DS_Store` is binary but lists the directory's filenames; parse it to discover unreferenced files, then fetch them:

```bash
python3 ds_store_exp.py https://target/.DS_Store
```

A served `.htpasswd` yields hashes to crack offline; a served `.htaccess` reveals rewrite rules, auth scope, and handler mappings that inform the [Apache handler](../apache/handler-and-type-mapping.md) and [Apache alias/rewrite](../apache/alias-and-rewrite-traversal.md) attacks.

## Exploitation

- Treat a recovered `APP_KEY`/`SECRET_KEY`/`machineKey` as game over for signed artifacts: forge sessions, cookies, JWT (when HMAC with that secret), and ViewState.
- DB credentials in `.env` often work against an exposed or tunnelable database port; API/cloud tokens pivot off-host.
- Dependency manifests (`composer.lock`, `package.json`) pin exact versions, which you map to known vulnerable-component attacks in the runtime and app layers.

## Tools

- **ffuf**/**feroxbuster** with dotfile wordlists; **ds_store_exp** for `.DS_Store`; **nuclei** exposure templates.

## References

- OWASP WSTG: Testing for information leakage; review webserver metafiles
- SecLists: common dotfile and config wordlists
