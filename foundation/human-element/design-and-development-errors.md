---
title: "Design and development errors: flaws built into the system"
description: "Security weaknesses introduced while a system is designed and built, from flawed logic and weak defaults to overlooked edge cases."
keywords:
  - design and development errors
  - software security flaws
  - secure design
  - edge cases
  - input validation
  - secure coding
---

# Design and development errors

**Design and development errors** are the security weaknesses baked into a system while it is being planned and built, before anyone misuses it or misconfigures it in production. They are not mistakes of operation; they are mistakes of construction. A flawed assumption in an architecture diagram, a piece of logic that handles the common case but not the strange one, or a convenient default that leaves a door open all become part of the shipped product and travel with it to every install.

## Flaws in the blueprint versus bugs in the code

It helps to separate two layers where builders go wrong:

- **Design flaws** live in the plan. The system was never meant to check something it should have checked, such as whether the person resetting a password actually owns the account. OWASP puts it bluntly: an insecure design cannot be fixed by a perfect implementation, because the missing control was never specified.
- **Implementation bugs** live in the code. The design was sound, but a developer forgot a length check, compared strings the wrong way, or pasted user input into a database query.

The distinction matters because the remedies differ. A bug can often be patched in a few lines. A flaw, such as trusting input that should never be trusted, usually means reworking how components talk to each other, which is slow and risky once the product is in customers' hands.

Both kinds share one property that sets them apart from [misconfiguration](misconfiguration.md) or [user misuse](user-behavior-and-misuse.md): they ship with the product. One weak default or logic error reaches every customer who runs the code, so a single mistake is multiplied across the whole user base.

## Six assumptions that turn into weaknesses

Most construction errors start as a reasonable-sounding assumption that nobody wrote down or challenged.

| Error | What the builder assumed | What actually happens |
| --- | --- | --- |
| Missing or weak input validation | Data arrives in the expected shape and size | Malformed or hostile input slips through and reaches logic that cannot handle it |
| Overlooked edge cases | Values will be ordinary | Empty fields, huge numbers, concurrent requests or unusual character encodings behave in ways no one tested |
| Insecure defaults | Users will harden the product after install | Open permissions, a sample password or disabled encryption stay that way, because most users never change them |
| Flawed trust between components | The internal service calling me is friendly | A compromise in one component spreads freely to every component that trusts it |
| Homegrown authentication logic | Login and sessions are simple enough to write ourselves | Custom code mishandles expiry, comparison or error states while looking correct |
| Mixing data and instructions | User text is just data | Text pasted directly into a query or command changes what the program does |

MITRE's Common Weakness Enumeration catalogs hundreds of these patterns in detail and is the usual vocabulary for naming them precisely.

## A password reset feature, built four ways wrong

Consider a single feature to see how both layers of error stack up. A team needs to let users reset forgotten passwords.

1. **The design flaw.** The spec says "user enters their email, then chooses a new password on the next screen." Nothing ties the second screen to proof of email ownership, so anyone who knows an address can take over the account. No amount of careful coding fixes this; the step is missing from the plan.
2. **The edge case.** After the design is corrected to email a reset link, the token never expires and is not invalidated after use. An old link found in a mailbox months later still works.
3. **The weak default.** The token is generated with a predictable value, because the convenient random function in the language was not designed for security. An attacker can guess valid tokens.
4. **The injection bug.** The email lookup builds its database query by concatenating the submitted address, so a crafted "email" rewrites the query.

Each step is a different kind of error, introduced at a different moment, and each would have been far cheaper to catch on paper or in review than after release, when it becomes a patch cycle, a coordinated disclosure and often an incident.

## Catching errors at the stage they are made

The most effective defenses line up with the stages where errors enter, an idea at the heart of [security by design](../principles/security-by-design.md) and of CISA's Secure by Design initiative, which asks manufacturers to own the security outcomes of what they ship.

- **While designing: threat model.** Ask what could go wrong while the design is still a diagram and changes cost a conversation. This is where the missing ownership check in the reset example gets caught.
- **While choosing building blocks: reuse proven components.** Prefer well-reviewed libraries for authentication, cryptography and parsing over writing your own.
- **While writing code: validate at every boundary.** Treat all external data as untrusted, check it against a strict expected format, and keep data separate from instructions (for example, parameterized queries).
- **While packaging: ship secure defaults.** Turn on the safest configuration out of the box and make users opt into anything riskier.
- **Before release: test for abuse, not only use.** Combine code review, static analysis and tests that deliberately feed the system bad, oversized and unexpected input.

People make these errors, so people are also part of the fix: developers who know the common weakness patterns are far less likely to introduce them (see [training and awareness](training-and-awareness.md)). And because some errors will always slip through, [least privilege](../principles/least-privilege.md) limits how much damage any single flaw can do.

## References

- [OWASP Top Ten](https://owasp.org/www-project-top-ten/)
- [OWASP Top Ten 2021, A04: Insecure Design](https://owasp.org/Top10/A04_2021-Insecure_Design/)
- [MITRE CWE: Common Weakness Enumeration](https://cwe.mitre.org/)
- [NIST SP 800-160 Vol. 1 Rev. 1, Engineering Trustworthy Secure Systems](https://csrc.nist.gov/pubs/sp/800/160/v1/r1/final)
- [CISA: Secure by Design](https://www.cisa.gov/securebydesign)
- [OWASP Cheat Sheet Series: Input Validation](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
