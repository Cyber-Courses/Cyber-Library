---
title: "Structured log field injection: JSONL breakage, key spoofing, and downstream parser confusion"
description: User-controlled values in JSON or key-value logs that break parsers, duplicate keys, or spoof severity fields.
keywords:
  - log injection
  - JSON logging
---

# Structured log injection

## Context

Embedding `"` or `}` in a free-text field can **terminate** JSON early or add **synthetic keys** if the logger does not escape. Some aggregators trust **first** occurrence of `level` or `msg` in a line.

## Theory

Use a logging API that **serializes** objects with a real JSON encoder, not string templates.
