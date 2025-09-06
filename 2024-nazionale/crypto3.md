# OliCyber.IT 2024 - National Final

## [crypto] Keyless Signatures (0 risoluzioni)

Se non hai nessuna chiave non puoi rompere nulla... Vero?

`nc keyless.challs.olicyber.it 38300`

Author: Matteo Rossi <@mr96>

## Panoramica

La challenge espone un servizio con 3 opzioni possibili:
1. firmare qualcosa per un utente a scelta, eccetto l'utente `admin`
2. verificare se stessi, fornendo il proprio nome utente e una firma di `Verifying myself: I am {user} on OliCyber.IT`, dove `user` è il nome utente scelto
3. inviare un messaggio a un utente scelto; questo restituirà la risposta dell'utente, che è sempre `ACK`, e la corrispondente firma (con la chiave dell'utente destinatario)

L'obiettivo è verificarsi come `admin`.

_Nota: questa challenge ha (almeno) 3 soluzioni possibili trovate durante lo sviluppo e il testing. Le riporteremo tutte e 3 nel writeup per completezza._

## Soluzione 1 (soluzione intended)
Analizziamo la funzione di verifica: data una firma $(s_1, s_2)$, verifica se

$$
s_2^e \equiv h(u)s_1^{h(m)} \pmod{n}
$$

dove $u$ è il nome utente per cui il messaggio viene verificato, $m$ è il messaggio e $h$ è la funzione di hash `sha256`. Ignoriamo la funzione di firma per il momento, poiché non è necessaria per questa soluzione.

Dall'opzione 3 possiamo ottenere una firma di `ACK` dall'admin. Chiamiamo questa firma $(s_1, s_2)$. Utilizzando l'extended gcd algorithm, possiamo trovare due coefficienti $a$ e $b$ tali che

$$
a\cdot e - b\cdot h(m_t) = -h(m_a)
$$

dove $m_t$ è il messaggio target (quello di verifica per l'admin) e $m_a$ è semplicemente `ACK`. A questo punto prendiamo $s_1' = s_1^e$ e $s_2'=s_2s_1^a$. La nuova coppia è una firma valida per $m_t$. Infatti sostituendo nell'equazione di verifica vediamo che il controllo diventa

$$
(s_2s_1^a)^e \equiv h(u)s_1^{b\cdot h(m_t)} \pmod{n}
$$

che, portando il termine in $s_1$ a sinistra e utilizzando la nostra identità, possiamo vedere che è lo stesso controllo per $m_a$, quindi verifica correttamente per l'utente admin.

## Soluzione 2 (trovata durante il testing da @Devrar)
Per questa seconda soluzione, ci serve sapere come funziona la funzione di firma. Dato un messaggio $m$ e un utente $u$, prende un numero casuale $r$ e la firma è $(s_1, s_2) = (r^e, h(u)^d\cdot r^{h(m)})$, dove $d$ è l'esponente privato (come in RSA) della chiave pubblica $(N, e)$ del server.

### Recuperare $h(u)^d$
Dalla prima query possiamo ottenere firme per qualsiasi utente $u$ diverso da `admin` e qualsiasi messaggio $m$. Sia $m$ tale che $h(m) = 0 \pmod{e}$ e $(s_1, s_2) = (r^e, h(u)^d\cdot r^{h(m)})$; in questo caso possiamo recuperare il valore di $h(u)^d$ calcolando

$$
s_2\cdot s_1^{\frac{h(m)}{e}} \pmod{n}.
$$

### Recuperare il valore di $r$ dalla query 3
Sia $u$ un utente per cui abbiamo recuperato $h(u)^d$ come spiegato nel passaggio precedente. Possiamo ora utilizzare la terza query per recuperare un random $r$ utilizzato per la firma di `ACK`. Sia $(s_1, s_2) = (r^e, h(u)^d\cdot r^{h(m_a)})$. Poiché conosciamo $h(u)^d$, possiamo recuperare il valore di $r^{h(m_a)}$. Poiché $\gcd(e, h(m_a)) = 1$, utilizzando l'identità di Bézout possiamo trovare $a, b$ tali che $a\cdot e + b\cdot h(m_a) = 1$. Possiamo quindi calcolare

$$
(r^e)^a \cdot (r^{h(m_a)})^b \pmod{n} = r^{a\cdot e + b\cdot h(m_a)} = r \pmod{n} = r,
$$

poiché $r < n$.

### Rompere il generatore di numeri casuali
I numeri casuali $r$ sono ottenuti attraverso il modulo `random` di Python, che utilizza l'algoritmo Mersenne Twister, che non è crittograficamente sicuro. Pertanto, una volta recuperate abbastanza osservazioni di $r$ con il passaggio precedente, siamo in grado di recuperare lo stato del Mersenne Twister e prevedere i futuri valori di $r$.

### Creare la firma
Ora che conosciamo i valori di $r$ utilizzati nella query 3, possiamo semplicemente inviare un messaggio a `admin` e recuperare $h(u_a)^d$ (l'utente admin) dividendo $s_2$ per $r^{h(m_a)}$ (cosa possibile poiché conosciamo $r$).

A questo punto possiamo ottenere una firma per il messaggio $m_t$ selezionando un valore casuale $r$ e restituendo

$$
(s_1, s_2) = (r^e, h(u_a)^d\cdot r^{h(m_t)}).
$$

Infine verifichiamo la firma e otteniamo la flag.

## Soluzione 3 (trovata durante il testing da @PhiQuadro)

Questa soluzione è simile alla prima nella struttura, ma utilizza il fatto che conosciamo il nome utente dell'admin.

Risolviamo $a\cdot h(m_t)+1 = e\cdot b$. A questo punto abbiamo che, in maniera simile alla soluzione 1, $s_1 = h(u_a)^a \pmod{n}$ e $s_2 = h(u_a)^b \pmod{n}$ formano una firma valida.

## Exploit (per la soluzione 1)
```py
#!/usr/bin/env python3

import logging
import os
from pwn import remote
from hashlib import sha256

logging.disable()

def h(m):
    return int.from_bytes(sha256(m).digest(), "big")

def extended_gcd(aa, bb):
    lastremainder, remainder = abs(aa), abs(bb)
    x, lastx, y, lasty = 0, 1, 1, 0
    while remainder:
        lastremainder, (quotient, remainder) = remainder, divmod(lastremainder, remainder)
        x, lastx = lastx - quotient*x, x
        y, lasty = lasty - quotient*y, y
    return lastremainder, lastx * (-1 if aa < 0 else 1), lasty * (-1 if bb < 0 else 1)

HOST = os.environ.get("HOST", "localhost")
PORT = int(os.environ.get("PORT", 38300))

r = remote(HOST, PORT)

pk = r.recvline()
n = int(pk.split()[-2][1:-1])
e = 65537

r.recvuntil(b"> ")
r.sendline(b"3")
r.recvuntil(b": ")
r.sendline(b"test")
r.recvuntil(b": ")
r.sendline(b"admin")
r.recvlines(2)
sig = r.recvline()
s1 = int(sig.split()[-2][1:-1])
s2 = int(sig.split()[-1][:-1])

# a*e - b*h("I am...") = -h("ACK")

to_forge = h(b"Verifying myself: I am admin on OliCyber.IT")
base = h(b"ACK")

g, c1, c2 = extended_gcd(e, -to_forge)
c1 *= -base
c2 *= -base
s1f = pow(s1, c2, n)
s2f = (s2*pow(s1, c1, n)) % n

r.recvuntil(b"> ")
r.sendline(b"2")
r.recvuntil(b": ")
r.sendline(b"admin")
r.recvuntil(b": ")
r.sendline(str(s1f).encode())
r.recvuntil(b": ")
r.sendline(str(s2f).encode())
print(r.recvline())
```
