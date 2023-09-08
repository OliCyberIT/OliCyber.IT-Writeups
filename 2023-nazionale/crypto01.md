# OliCyber.IT 2023 - Competizione Nazionale

## [crypto] Geroglifici (84 risoluzioni)

La challenge mostra la flag cifrata con delle emoji, e permette di richiedere di cifrare del testo arbitrario.

### Soluzione

La cifratura usata è monoalfabetica, quindi è sufficiente conoscere quali emoji corrispondono a quali caratteri in chiaro per poter leggere la flag.

Chiedendo come testo da cifrare l'intero alfabeto è immediato ottenere questa corrispondenza.

### Exploit

```python
HOST = os.environ.get("HOST", "localhost")
PORT = int(os.environ.get("PORT", 35000))

r = remote(HOST, PORT)
r.recvlines(3)
data = r.recvline(False)
flag_enc = data.decode("utf-8").strip()[20:]
alphabet = string.ascii_letters + string.digits + '_{}!'
r.sendlineafter("> ", alphabet)
resp = r.recvline(False).decode("utf-8")
code = {y: x for x, y in zip(alphabet, resp)}
flag = ""
for c in flag_enc:
    flag += code[c]
print(flag)
```
