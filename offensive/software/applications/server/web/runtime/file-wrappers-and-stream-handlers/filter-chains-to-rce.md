---
title: "PHP filter chains to RCE: synthesizing a payload from conversion filters"
order: 2
description: "Turning a php://filter read primitive into remote code execution by chaining convert.iconv filters to generate arbitrary bytes, without needing any file on disk."
keywords:
  - php filter chain
  - convert.iconv
  - LFI to RCE
  - filter chain oracle
  - no file RCE
---

# Filter chains to RCE

A `php://filter` read primitive looks like disclosure-only, but chained **conversion filters can generate arbitrary bytes from nothing**. By stacking many `convert.iconv.*` steps, an attacker makes the wrapper emit a chosen PHP string at the front of the stream, so a sink that *executes* the wrapper's output (such as an `include`) runs attacker code even though no attacker-controlled file exists on disk. This upgrades many read-only LFI sinks to full RCE.

## The idea

`convert.iconv.<from>.<to>` transforms text between encodings. Certain encoding pairs reliably prepend or mutate bytes, and some error out on specific byte values. Composing these:

- lets you **append controlled bytes** to the data (building up a target string character by character), and
- provides an **oracle**: a chain that errors (or not) depending on the leading byte, which is enough to brute-force content when you only have a read.

Stacked far enough, the chain produces a valid `<?php ... ?>` payload as the stream's leading bytes. When the sink includes that stream, the payload executes.

## Using it

The chains are long and mechanical, so generate them with a tool rather than by hand. **php_filter_chain_generator** builds a ready `php://filter/...` string for a chosen command:

```bash
python3 php_filter_chain_generator.py --chain '<?php system($_GET["c"]); ?>'
# emits: php://filter/convert.iconv.UTF8.CSISO2022KR|...|convert.base64-decode/resource=php://temp
```

Place the emitted wrapper in the vulnerable parameter (for example `?page=<chain>`), then drive the command through the smuggled payload (`&c=id`). The `resource=` can point at a trivially readable or empty stream because all the content comes from the filters.

## Exploitation notes

- The sink must *execute* the stream (`include`/`require`/`eval` of file contents). A pure `file_get_contents` that only echoes gives disclosure, not execution, though the same chain technique can still fabricate content for other sinks.
- No `allow_url_include`, no uploaded file, and no writable directory are required, which is what makes this powerful against hardened LFI where traditional log-poisoning or upload paths are closed.
- The chains are large; if a WAF caps parameter length, this vector may be impractical and phar or data wrappers are the fallback.

## Tools

- **php_filter_chain_generator** (synacktiv) and equivalents.

## References

- Synacktiv: PHP filter chains research
- PHP manual: convert.iconv filters
