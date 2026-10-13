---
order: 2
title: "Désérialisation pickle Python : __reduce__ comme primitive de RCE directe"
description: "Exploiter la désérialisation Python : pickle.loads sur des données non fiables exécute __reduce__ pour une exécution de code immédiate, plus les sinks yaml.load et jsonpickle."
keywords:
  - pickle python
  - __reduce__
  - yaml.load
  - jsonpickle
  - désérialisation
---

# Pickle Python

Le module `pickle` de Python est non sécurisé par conception pour les entrées non fiables : le format est un petit langage à pile que l'unpickler exécute, et un objet peut déclarer, via `__reduce__`, un callable et des arguments à exécuter au moment du chargement. Contrairement à PHP ou Java, il n'y a pas de chasse à la chaîne de gadgets ; `pickle.loads()` sur des octets de l'attaquant est une **exécution de code directe et immédiate**.

## La primitive __reduce__

`__reduce__` renvoie un callable et un tuple d'arguments ; l'unpickler l'appelle pendant la reconstruction. Une classe de payload nomme `os.system` (ou `subprocess`, `builtins.eval`, `builtins.exec`) comme callable :

```python
import pickle, os, base64

class RCE:
    def __reduce__(self):
        return (os.system, ('id',))

payload = base64.b64encode(pickle.dumps(RCE()))
print(payload.decode())
```

Tout ce qui appelle ensuite `pickle.loads(base64.b64decode(payload))` exécute `id`. Sinks courants : blobs de session ou de cache, valeurs `Authorization`/cookie, payloads de file de messages, fichiers de modèle ML, et toute API qui accepte un objet picklé. La même chose vaut pour `cPickle`, `shelve`, `dill` et `joblib`, qui enveloppent pickle.

Reconnaissez pickle sur le réseau à ses opcodes et à son `.` final (le protocole 0 est quasi-ASCII ; les protocoles supérieurs sont binaires et souvent encodés en base64).

## Autres sinks Python

- **`yaml.load`** (PyYAML) sans loader sûr construit des objets Python arbitraires via le tag `!!python/object/apply` :

  ```yaml
  !!python/object/apply:os.system ["id"]
  ```

  `yaml.load(data)` exécute cela sous `yaml.Loader`/`UnsafeLoader` (et le `yaml.load` par défaut d'avant 5.1 sans loader). `FullLoader` a été spécifiquement conçu pour rejeter l'application de callable arbitraire et ne l'exécute pas par défaut ; il n'était exploitable que via des contournements historiques précis que PyYAML actuel a corrigés. `yaml.safe_load` ne l'exécute jamais. Testez à la fois le sink et le loader exact utilisé.

- **`jsonpickle.decode`** reconstruit des objets Python à partir de JSON qui porte des clés `py/object` et `py/reduce`, donnant la même primitive `__reduce__` via un payload d'apparence JSON.

- **`numpy.load(allow_pickle=True)`, `pandas.read_pickle`, `joblib.load` et `torch.load`** désérialisent avec `pickle` en coulisses, donc charger un fichier `.npy`/`.pkl`/de modèle non fiable est la même primitive.

## Notes d'exploitation

- Gardez les imports du payload minimaux et présents sur la cible (`os`, `subprocess`, `builtins`) ; pour la sortie, utilisez `subprocess.check_output` et exfiltrez, ou visez un reverse shell puisque la sortie d'`os.system` ne vous est pas renvoyée.
- Pour les environnements restreints qui bloquent `os`, `builtins.eval`/`exec` avec une chaîne construite atteint le même objectif.
- `pickle` s'exécute avant que tout code applicatif n'inspecte l'objet, donc la validation d'entrée après `loads()` arrive trop tard et est sans effet sur l'exploitation.

## Outils

- Payloads `__reduce__` forgés à la main ; **pker** pour assembler des opcodes pickle complexes.

## Références

- Python manual: pickle (security warning)
- PortSwigger Web Security Academy: Insecure deserialization
