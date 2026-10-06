---
title: "File upload abuse"
order: 1
description: "Turning an upload feature into code execution or file overwrite by defeating the checks on name, type, content, and destination of attacker-supplied files."
keywords:
  - file upload
  - web shell
  - unrestricted upload
  - zip slip
  - arbitrary file write
---

# Upload

Upload abuse starts from any feature that writes attacker-supplied bytes to disk: avatars, document attachments, import wizards, profile backgrounds.

The prize is control over three properties of the stored file: its **name and extension** (so the server executes it), its **declared and real content** (so it passes type checks yet still runs), and its **destination path** (so it lands where it is served or where it overwrites something important). Unrestricted upload chains those first two to drop a web shell into a web-served directory and then requests it. Archive extraction (Zip Slip) attacks the third: entry names inside a crafted archive escape the unpack directory to write wherever the process can, overwriting assets, binaries, or configuration. Both routes commonly end in remote code execution.

## Pages

- **[Unrestricted upload](unrestricted-upload.md)**: Defeating extension, Content-Type, and magic-byte checks to store an executable script in a web-served path, then requesting it for remote code execution.
- **[Zip Slip](zip-slip.md)**: Archive entries with traversal sequences or absolute paths write outside the unpack directory during extraction, overwriting web assets or binaries and plant...

## Tools

- **[Burp Suite](https://portswigger.net/burp)**: Intruder for fuzzing extensions, Content-Type, and magic bytes.
- **[exiftool](https://exiftool.org/)**: embed payloads in image metadata to build polyglots.

## References

- [OWASP: Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [PayloadsAllTheThings: Upload Insecure Files](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files)
