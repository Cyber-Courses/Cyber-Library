---
title: "Python pickle deserialization: __reduce__ as a direct RCE primitive"
description: "Exploiting Python deserialization: pickle.loads on untrusted data runs __reduce__ for immediate code execution, plus yaml.load and jsonpickle sinks."
keywords:
  - python pickle
  - __reduce__
  - yaml.load
  - jsonpickle
  - deserialization
---

# Python pickle

Python's `pickle` module is unsafe by design for untrusted input: the format is a small stack language that the unpickler executes, and an object can declare, via `__reduce__`, a callable and arguments to run at load time. Unlike PHP or Java, there is no gadget-chain hunt; `pickle.loads()` on attacker bytes is **direct, immediate code execution**.

## The __reduce__ primitive

`__reduce__` returns a callable and a tuple of arguments; the unpickler calls it during reconstruction. A payload class names `os.system` (or `subprocess`, `builtins.eval`, `builtins.exec`) as the callable:

```python
import pickle, os, base64

class RCE:
    def __reduce__(self):
        return (os.system, ('id',))

payload = base64.b64encode(pickle.dumps(RCE()))
print(payload.decode())
```

Anything that then calls `pickle.loads(base64.b64decode(payload))` runs `id`. Common sinks: session or cache blobs, `Authorization`/cookie values, message-queue payloads, ML model files, and any API that accepts a pickled object. The same applies to `cPickle`, `shelve`, `dill`, and `joblib`, which wrap pickle.

Recognize pickle on the wire by its opcodes and trailing `.` (protocol 0 is ASCII-ish; higher protocols are binary and often base64-encoded).

## Other Python sinks

- **`yaml.load`** (PyYAML) without a safe loader constructs arbitrary Python objects via the `!!python/object/apply` tag:

  ```yaml
  !!python/object/apply:os.system ["id"]
  ```

  `yaml.load(data)` on older PyYAML (or with `Loader=yaml.Loader`/`FullLoader` gaps) executes it; `yaml.safe_load` does not. Test both the sink and the loader in use.

- **`jsonpickle.decode`** reconstructs Python objects from JSON that carries `py/object` and `py/reduce` keys, giving the same `__reduce__` primitive through a JSON-looking payload.

- **`feedparser`, `numpy.load(allow_pickle=True)`, and various caching layers** reach `pickle` internally.

## Exploitation notes

- Keep the payload's imports minimal and present on the target (`os`, `subprocess`, `builtins`); for output, use `subprocess.check_output` and exfiltrate, or go for a reverse shell since `os.system` output is not returned to you.
- For restricted environments that block `os`, `builtins.eval`/`exec` with a constructed string reaches the same place.
- `pickle` executes before any application code inspects the object, so input validation after `loads()` is too late and irrelevant to exploitation.

## Tools

- Hand-crafted `__reduce__` payloads; **pker** for assembling complex pickle opcodes.

## References

- Python manual: pickle (security warning)
- PortSwigger Web Security Academy: Insecure deserialization
