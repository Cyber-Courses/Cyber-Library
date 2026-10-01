---
title: "HTTP and HTTPS: the baseline SSRF scheme"
description: "The default http and https schemes reach internal web services, health and debug endpoints, and the cloud metadata API, and are the starting point before escalating to other schemes."
keywords:
  - http SSRF
  - https SSRF
  - cloud metadata
  - internal web service
  - 169.254.169.254
---

# HTTP and HTTPS

`http` and `https` are the schemes a fetch sink is built for, and most SSRF value is reachable without ever changing scheme. The attacker simply redirects the request at an internal web target instead of the intended external one.

## Internal web targets

Any web service the server can reach is in scope, including those bound to loopback or an internal network and never exposed publicly:

```
http://127.0.0.1:8080/admin
http://localhost:8161/            # internal management consoles
http://10.0.0.5/actuator/env      # framework debug and management endpoints
http://internal-api.corp/v1/keys
```

Management and debug endpoints are high value because they are often unauthenticated when reached from localhost, and they expose configuration, environment variables, and credentials.

## The cloud metadata service

On a cloud instance, the link-local metadata endpoint returns instance identity and, frequently, temporary credentials:

```
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/user-data/
```

Where the response is reflected, the credential document comes straight back. Equivalents exist across providers at the same `169.254.169.254` address and at provider-specific hostnames, sometimes requiring a specific header, which matters when the sink lets the attacker influence headers.

## Role in the scheme tree

HTTP is the baseline and the first thing to try, but it is a read of whatever the service returns. When a discovered service needs a crafted multi-line request or a raw command (Redis, SMTP, a `POST` with a precise body), escalate to [Gopher](gopher.md). For local files, switch to [File](file.md). Pair HTTP probing with [Port](../authority/port.md) scanning to find the services worth aiming a scheme payload at, and with the [Authority](../authority/index.md) encodings when the internal address is filtered.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
