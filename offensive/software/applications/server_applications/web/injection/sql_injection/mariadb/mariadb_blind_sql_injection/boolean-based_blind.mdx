---
title: Boolean-based Blind SQL Injection Explained
description: Learn how to exploit Boolean-based Blind SQL Injection in MariaDB for
  secure data extraction through conditional logic.
keywords:
- Boolean-based Blind
- SQL Injection
- MariaDB
- conditional logic
- data extraction
- Boolean logic
- pattern matching
- ASCII function
- SUBSTRING function
- REGEXP binary match
- HTTP response analysis
- injection context
- WAF bypass
---

# Boolean-based Blind SQL Injection

## Context

Boolean-based Blind SQL Injection is a technique used in the field of offensive cybersecurity to exploit vulnerabilities in applications using MariaDB (or similar databases), enabling unauthorized data extraction. This method leverages the application's responses to specially crafted SQL queries that result in true or false conditions, even when direct database responses are not visible to the attacker. Mastery of SQL query structure, Boolean logic, conditional statements, and prior knowledge of MariaDB Blind SQL Injection techniques are assumed for effective execution of this attack.

## Theory

### Boolean-based Blind SQL Injection

Boolean-based Blind SQL Injection exploits the application's behavior when a SQL query returns a true or false response. An attacker can infer sensitive data by systematically manipulating the query conditions to validate his hypotheses about the database's contents.

### Conditional Logic in SQL

At the core of Boolean-based Blind SQL Injection is the utilization of SQL conditional logic. Attackers craft queries that leverage conditional statements such as `IF()`. By monitoring whether the application returns a change in its response or behavior (such as success or error), attackers can determine if a condition is true or false.

### Functions for Data Extraction

To retrieve data, attackers can use SQL functions like `SUBSTRING`, `ASCII`, and `IF()`. The `SUBSTRING` function allows selection of specific portions of a string, enabling byte-by-byte extraction of data. Combined with `ASCII`, it allows determination of specific character values by their numeric equivalents.

### Pattern Matching Techniques

Pattern matching functions such as `LIKE` and `REGEXP` can infer data structure and content by matching specific patterns within the data. These can be used during SQL Injection to understand bits of the data through pattern representation, especially when used to decode structures hidden in blind attacks by matching certain segments or characters.

### Prerequisites for Boolean-based Blind Injection

For successful boolean-based blind injection, several conditions must be met:
- **MariaDB/MySQL version**: Compatible with IF(), SUBSTRING(), ASCII() functions
- **Error handling**: Application must return different responses for true/false conditions
- **Injection context**: Vulnerable parameter in URL, form, or HTTP header
- **Response analysis**: Ability to distinguish between different HTTP responses (status codes, content length, error messages)

### Response Analysis in Blind Injection

Boolean-based blind injection relies on analyzing application responses to determine if conditions are true or false:
- **True condition**: Normal page content, successful HTTP status (200), expected response length
- **False condition**: Error page, different content, HTTP error status (404/500), different response length
- **Baseline**: Establish normal response pattern before testing conditions

## Practice

### Boolean-based Blind SQL Injection

The following steps outline a manual execution of Boolean-based Blind SQL Injection to extract sensitive data from a database using realistic injection contexts.

- **Establish Baseline Response**: First, test a normal request to establish the baseline response pattern.

    ```http
    http://example.com/page?id=1
    ```

    Note the HTTP status code, response length, and content to establish what a "normal" response looks like.

- **Test Basic Injection**: Inject a simple true condition to verify the injection point works.

    ```http
    http://example.com/page?id=1' AND 1=1--
    ```

    If this returns the same response as the baseline, the injection is working. If `1=1` returns different content than `1=2`, you have a boolean-based blind injection.

- **Character-by-Character Extraction**: Extract data character by character using conditional logic.

    ```http
    http://example.com/page?id=1' AND IF(SUBSTRING((SELECT password FROM users WHERE id=1),1,1)='a',1,0)--
    ```

    This payload checks if the first character of the password is 'a'. Compare the response with your baseline to determine if the condition is true or false.

- **ASCII Range Testing**: Use ASCII values to systematically test character ranges.

    ```http
    http://example.com/page?id=1' AND ASCII(SUBSTRING((SELECT password FROM users WHERE id=1),1,1))>97--
    ```

    Test ranges like 97-122 (lowercase letters), 65-90 (uppercase), 48-57 (numbers) to narrow down the character.

- **Pattern Matching with LIKE**: Use LIKE for pattern-based extraction.

    ```http
    http://example.com/page?id=1' AND (SELECT password FROM users WHERE id=1) LIKE 'a%'--
    ```

    This checks if the password starts with 'a'. Note the corrected syntax: `column LIKE pattern`, not `pattern LIKE column`.

- **Iterative Character Extraction**: Continue the process for each character position.

    ```http
    http://example.com/page?id=1' AND IF(SUBSTRING((SELECT password FROM users WHERE id=1),2,1)='b',1,0)--
    ```

    Move to the second character, third character, etc., until you have extracted the complete password.

- **Response Analysis**: Carefully analyze each response to determine true/false conditions.

    - **True condition**: Same response as baseline (normal page content, HTTP 200)
    - **False condition**: Different response (error page, HTTP 404/500, different content length)
    - **Use tools**: Burp Suite's response comparison, custom scripts for automated analysis

The outcome of these steps is the incremental access to sensitive data stored in the database by iteratively using Boolean logic to confirm each character or piece of data through HTTP response analysis.

## Tools

- **sqlmap**: Powerful automated tool for detecting and exploiting boolean-based blind SQL injection vulnerabilities, with built-in support for MariaDB-specific functions and response analysis.
- **Burp Suite**: Comprehensive platform for manual testing, essential for analyzing HTTP responses, comparing response lengths, and identifying true/false conditions in boolean-based blind injection.
- **OWASP ZAP**: Alternative web application security scanner that can assist in identifying boolean-based blind injection vulnerabilities.
- **Custom Python scripts**: For automated response analysis, character extraction, and systematic testing of ASCII ranges.

## Common Challenges and Solutions

### WAF Bypass Techniques
- Use alternative functions: `MID()` instead of `SUBSTRING()`, `ORD()` instead of `ASCII()`
- Obfuscate keywords: `SEL/**/ECT`, `UNI/**/ON`
- Use hex encoding: `0x61646d696e` instead of `'admin'`
- Case variation: `SeLeCt` instead of `SELECT`

### Response Analysis Challenges
- **Network latency**: Test multiple times and account for timing variations
- **Dynamic content**: Look for consistent differences, not absolute values
- **Error suppression**: Some applications suppress errors; look for subtle content differences
- **Rate limiting**: Use delays between requests to avoid triggering rate limits

### Optimization Techniques
- **Binary search**: Use ASCII ranges (e.g., `>97` then `<110`) to reduce requests
- **Common patterns**: Test common password patterns first (e.g., dictionary words)
- **Parallel testing**: Test multiple character positions simultaneously when possible
- **Automation**: Use scripts to automate the tedious character-by-character extraction

By mastering these techniques, you can effectively extract sensitive data from MariaDB databases using boolean-based blind SQL injection, while understanding the real-world challenges and constraints of this attack method.
