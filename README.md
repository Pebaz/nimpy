# nimpy [![Build Status](https://travis-ci.org/yglukhov/nimpy.svg?branch=master)](https://travis-ci.org/yglukhov/nimpy)

Native language integration with Python has never been easier!

## Implementing a Python Module in Nim
```nim
# mymodule.nim - file name should match the module name you're going to import from python
import nimpy

proc greet(name: string): string {.exportpy.} =
    return "Hello, " & name & "!"
```

```bash
# Compile on Windows:
$ nim c --threads:on --app:lib --out:mymodule.pyd mymodule
# Compile on everything else:
$ nim c --threads:on --app:lib --out:mymodule.so mymodule
```

```py
# test.py
import mymodule
assert mymodule.greet("world") == "Hello, world!"
assert mymodule.greet(name="world") == "Hello, world!"
```

## Calling Python From Nim
```nim
import nimpy
let os = pyImport("os")
echo "Current dir is: ", os.getcwd().to(string)

# sum(range(1, 5))
let py = pyBuiltinsModule()
let s = py.sum(py.range(0, 5)).to(int)
assert s == 10
```
Note: here nimpy relies on your local python installation.


## Importing Nim Extensions Directly

For a convenient way to import your Nim extension modules directly, you can use
[Nimporter](https://github.com/Pebaz/Nimporter). It scans Python's path to find
Nim files and compiles them when imported. Subsequent imports utilize a cached
version of the module so you must only pay for the compilation once. Changes to
the Nim source files trigger a recompilation since Nimporter maintains a hash of
the source. Both build artifacts and hashed sources are kept in the
`__pycache__` directory so that the built `.pyd` or `.so` files do not clog up
the filesystem. In addition, since the `__pycache__` directory is usually
ignored via `.gitignore` for example, you can transparently use Nim code without
any real impediments.

To use Nimporter, just install via Pip:

```bash
$ pip3 install nimporter
```

Usage example:

```nim
# foo.nim
import nimpy

proc greet(name: string): string {.exportpy.} =
    return "Hello, " & name & "!"
```

Now import and use the file without an explicit build step:

```python
import nimporter  # Now any Nim file on the path can be directly imported.
import foo  # foo.nim is built, cached, and hashed

print(foo.greet('Pebaz'))
```

## Misc
The library is designed with ABI compatibility in mind. That is
the compiled module doesn't depend on particular Python version, it should
properly work with any. The C API symbols are loaded in runtime from whichever
process has launched your module.


## Troubleshooting, Q&A
<details>
<summary> <b>Question:</b>

Importing the compiled module from Python fails with `ImportError: dynamic module does not define module export function ...`
</summary>

  Make sure that the module you're importing from Python has exactly the same name as the `nim` file which the module is implemented in.
</details>

<details>
<summary> <b>Question:</b>

Nim strings are converted to Python `bytes` instead of `string`
</summary>

  nimpy converts Nim strings to Python strings usually, but since Nim strings are encoding agnostic and may contain invalid utf8 sequences, nimpy will fallback to Python `bytes` in such cases.
</details>

<details>
<summary> <b>Question:</b>

Is there any numpy compatibility?
</summary>

  nimpy allows manipulating numpy objects just how you would do it in Python,
however it not much more efficient. To get the maximum performance nimpy
exposes [Buffer protocol](https://docs.python.org/3/c-api/buffer.html), see
[raw_buffers.nim](https://github.com/yglukhov/nimpy/blob/master/nimpy/raw_buffers.nim).
[tpyfromnim.nim](https://github.com/yglukhov/nimpy/blob/master/tests/tpyfromnim.nim)
contains a very basic test for this (grep `numpy`). Higher level API might
be considered in the future, PRs are welcome.
</details>

<details>
<summary> <b>Question:</b>

Does nim default garbage collector (GC) work?
</summary>

  nimpy internally does everything needed to run the GC properly (keeps the stack bottom
  actual, and appropriate nim references alive), and doesn't introduce any special rules
  on top. So the GC question boils down to proper GC usage in nim shared libraries,
  you'd better lookup elsewhere. The following guidelines are by no means comprehensive,
  but should be enough for the quick start:
  - If it's known there will be only one nimpy module in the process, you should be fine.
  - If there is more than one nimpy module, it is recommended to [move nim runtime out
    to a separate shared library](https://nim-lang.org/docs/nimc.html#dll-generation).
    However it might not be needed if nim references are known to never travel between
    nim shared libraries.
  - If you hit any GC problems with nimpy, whether you followed these guidelines or not,
    please report them to nimpy tracker :)

</details>

## Future directions
* exporting Nim types/functions as Python classes/methods
* High level buffer API

## Stargazers over time

[![Stargazers over time](https://starcharts.herokuapp.com/yglukhov/nimpy.svg)](https://starcharts.herokuapp.com/yglukhov/nimpy)
