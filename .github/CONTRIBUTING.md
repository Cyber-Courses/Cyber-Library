# Contributing to Cyber Library

Thanks for helping. Cyber Library is plain Markdown, published to [cyberlibrary.com](https://www.cyberlibrary.com) in 70 languages. Every page sits in one shared taxonomy, so a good contribution is usually small, precise, and in the right place.

There are four ways to help:

1. [Propose a topic](#propose-a-topic)
2. [Write or fix a page](#write-or-fix-a-page)
3. [Translate](#translate)
4. [Report a problem](#report-a-problem)

New here? Pick an issue labelled [`good first issue`](https://github.com/Cyber-Courses/Cyber-Library/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22) or [`translation`](https://github.com/Cyber-Courses/Cyber-Library/issues?q=is%3Aissue+is%3Aopen+label%3Atranslation). Questions go to [Discord](https://discord.gg/a9XwRKxdHf).

The [wiki](https://github.com/Cyber-Courses/Cyber-Library/wiki) has the longer versions: [Philosophy of the content](https://github.com/Cyber-Courses/Cyber-Library/wiki/Philosophy-of-the-Content), [Writing style guide](https://github.com/Cyber-Courses/Cyber-Library/wiki/Writing-Style-Guide), [File structure](https://github.com/Cyber-Courses/Cyber-Library/wiki/File-Structure) and [Internationalization](https://github.com/Cyber-Courses/Cyber-Library/wiki/Internationalization-(i18n)).

## How the repo is organized

```text
foundation/  offensive/  defensive/  governance/  intelligence/  career/   English content, one folder per track
i18n/<lang>/                                                              translations, mirroring the English paths
```

- A folder with an `index.md` is a **section**; any other `<slug>.md` is a **page**.
- Folder and file names are lowercase, hyphenated, and always in English, also inside `i18n/`.
- The folder tree is the site navigation: the path of a file is its place in the sidebar and breadcrumb.
- The whole taxonomy, published and planned, is browsable on the [structure map](https://www.cyberlibrary.com/en/structure).

## Propose a topic

Every page is planned as a topic in the taxonomy before it is written, so the tree stays coherent.

1. Search the [site](https://www.cyberlibrary.com) (press <kbd>⌘</kbd> <kbd>K</kbd>) and the [structure map](https://www.cyberlibrary.com/en/structure): the topic may already exist or be planned.
2. Open a [new topic request](https://github.com/Cyber-Courses/Cyber-Library/issues/new?template=new-topic-request.yml) with the title, the track, and where in the tree you think it belongs.
3. A maintainer places it in the taxonomy and replies with the final path and slug. You can then write it.

## Write or fix a page

### Workflow

```bash
# 1. Fork Cyber-Courses/Cyber-Library on GitHub, then:
git clone https://github.com/<you>/Cyber-Library.git
cd Cyber-Library
git remote add upstream https://github.com/Cyber-Courses/Cyber-Library.git

# 2. Branch from dev (not main: main is what the website serves)
git fetch upstream
git checkout -b content/<short-topic> upstream/dev

# 3. Edit, commit, push
git commit -am "content: <what you changed>"
git push origin content/<short-topic>
```

4. Open a pull request **into `dev`** and fill in the template checklist.

### House style

These rules are checked in review. Most rejected pull requests miss one of them.

- **Frontmatter**: YAML with `title`, `description` and `keywords` (a list), consistent with the body. `title` is the SEO title and can be descriptive.
- **H1 is the leaf concept only.** The breadcrumb already shows the parents: under `dacl/` the page is `# Targeted Kerberoasting`, under `smtp/` it is `# Smuggling`, not `# SMTP smuggling`. No backticks in the H1 or `title`.
- **Offensive pages explain and exploit, nothing else.** No detection, mitigation, remediation or prevention sections: those live in the Defensive track. End with `## Tools` (optional) and `## References`.
- **Deep, not wide.** Show the mechanism (the field, flag, API or protocol detail that makes it work), the preconditions to check first, real worked commands in tagged code fences with what the output means, and the variants. Never list tool names instead of showing the command.
- **Payloads must be correct** for the exact engine, shell or version you name. Test them or reason them through end to end.
- **References are real, clickable links** to primary sources: vendor docs, RFCs, original research, tool repos.
- **No em dashes or en dashes** in prose or frontmatter. Use commas, colons, parentheses or two sentences. Write ranges as "2 to 3".
- **No CVE identifiers.** Teach the vulnerability class and its mechanism.
- **No `## See also`** and no scope or legal disclaimer blockquote: the site adds cross-links and the disclaimer.
- **American English.**
- **Sibling order**: siblings sort alphabetically. Add a numeric `order:` to the frontmatter only when the reading order matters (enumeration before exploitation, for example).
- **Section pages** (`index.md`) orient the reader and link every child page and subsection.

A minimal page:

````markdown
---
title: Targeted Kerberoasting with WriteSPN in Active Directory
description: Set a temporary SPN on a user you can write to, request its service ticket, crack it offline.
keywords:
- targeted kerberoasting
- writespn
- active directory
---

# Targeted Kerberoasting

Two or three sentences on what it is and why it works.

## The sequence

```bash
targetedKerberoast.py -v -d example.local -u user -p pass
```

## References

- [targetedKerberoast](https://github.com/ShutdownRepo/targetedKerberoast)
````

## Translate

Translations live in `i18n/<lang>/` and mirror the English paths file for file. Today they cover the track landing pages (`i18n/<lang>/<track>/index.md`).

1. Make sure the English page exists. Translate from it, never from another translation.
2. Copy it to the same path under `i18n/<lang>/`. Keep folder and file names in English.
3. Translate the frontmatter `title`, `description` and `keywords` and the body. Keep the Markdown structure, code blocks, commands and links unchanged.
4. Use the security meaning of each term, not the dictionary one. "Intelligence" is threat intelligence (renseignement, Aufklärung, 情报), not intellect. "Foundation" means fundamentals. "Offensive" is offensive security.
5. The house style applies: no em dashes, and keep the H1 short.
6. Open a pull request into `dev` with the `translation` label, or report a bad translation with the [translation issue](https://github.com/Cyber-Courses/Cyber-Library/issues/new?template=translation-issue.yml) template.

If your language has no folder yet, open an issue and a maintainer will create it.

## Report a problem

Use the [issue templates](https://github.com/Cyber-Courses/Cyber-Library/issues/new/choose): content correction (wrong payload, outdated tool, broken link), new topic, translation issue, website bug, or wiki change. Link the page URL and quote the exact line.

## Review flow

1. **Automated review.** Every pull request gets an automated code review that flags technical errors in payloads and commands. Address each comment or explain why it does not apply.
2. **Maintainer review.** A maintainer checks placement in the taxonomy, the house style, and technical accuracy, and labels the pull request (`content`, `new`, `improvement`, `translation`).
3. **Merge.** Approved pull requests are squash-merged into `dev`.
4. **Release.** `dev` is promoted to `main` in a numbered release ([V1.0.x](https://github.com/Cyber-Courses/Cyber-Library/releases)), and the website publishes it.

Thank you for making security knowledge easier to find.
