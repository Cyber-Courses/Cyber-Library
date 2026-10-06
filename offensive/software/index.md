---
title: "Software: offensive techniques against applications and operating systems"
description: "The software category covers attacks on code and the systems that run it, split between the applications that implement behavior and the operating systems that host them."
keywords:
  - software security
  - application security
  - operating system attacks
  - privilege escalation
  - exploitation
---

# Software

The software category covers offensive work against code and the systems that execute it, the largest and most actively developed part of the library. It spans flaws in the programs an organization runs and in the operating systems underneath them, from a logic error in a web request handler to a privilege-escalation path in a kernel.

## Why it is split this way

The two subcategories divide by what the code is and who controls it:

- **[Applications](applications/index.md)**: the programs that implement behavior, whether they run on a user's device or on a server. These flaws live in what developers wrote and in how they composed frameworks and libraries.
- **[Operating systems](operating-systems/index.md)**: the platforms that host applications, where attacks target the kernel, system services, drivers, and the privilege and isolation model that everything above depends on.

The split reflects a real boundary in both ownership and technique. Application flaws are usually fixed by changing the application's own code or configuration, and they are reached through the interfaces the application exposes. Operating-system flaws are fixed by the platform vendor or the system administrator, and they are reached through system calls, local privilege boundaries, and services. An assessment typically moves between the two, using an application foothold to reach the host and then the operating system to deepen control, but the techniques and the people who remediate them are distinct, which is why the category branches here first.

## References

- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [MITRE ATT&CK](https://attack.mitre.org/)
