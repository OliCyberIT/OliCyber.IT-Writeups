# OliCyber.IT 2022 - Competizione territoriale

## [binary-2] Rererererererecursive (10 risoluzioni)

La challenge stampa innanzitutto un bel Gabibbo in ASCII art, poi inizia a calcolare la flag utilizzando una funzione risorsiva chiamata `serie`.
Il calcolo non terminerà mai perché la funzione è ricorsiva e chiama sé stessa due volte, quindi il numero di chiamate è esponenziale.

### Soluzione

Anche eliminando la `sleep` dentro la funzione `gabibbo_work`, non otterremo niente di utile.
In particolare, essendo il numero di chiamate assolutamente enorme, è probabile che il programma segfaulti a causa di uno stack overflow.

Il modo per risolvere la challenge è con carta e penna: ci accorgiamo che quello che fa il programma è

- calcolare una chiave, un intero a 64 bit, con la funzione `serie`
- xorare la chiave con la flag

Essendo un OTP, è improbabile che qualche attacco crittografico possa funzionare, ma dato che abbiamo una ricetta completa per la chiave, possiamo andare a calcolarcela in parte e poi ricavare la flag.

La funzione che calcola la chiave, definisce la seguente successione

$$
\begin{cases}
    s_{i + 2} = 3 s_{i + 1} - 2 s_i \\
    s_0 = 0 \\
    s_1 = 1
\end{cases}
\qquad
k = s_1000000
$$

Posto che possiamo trasformare il calcolo per l'n-esimo termine della successione esattamente come si fa per la serie di Fibonacci, cioè usando la programmazione dinamica, vale la pena mostrare invece come calcolarlo con carta e penna, che probabilmente è un metodo che meno persone hanno già visto.

Il metodo è ben fondato, ma per essere usato con cognizione di causa, richiede un po' di teoremi che si fanno ad analisi 1, per cui alcuni punti potrebbero sembrare un po' oscuri. Riguardatelo fra qualche anno e tutto sarà cristallino.

Dato che l'equazione per la successione è lineare, è sensato andare a cercare soluzioni di tipo esponenziale, cioè $s_k = \lambda^k$:

$$
\lambda^{k + 2} = 3 \lambda^{k + 1} - 2 \lambda^k   \\
\rightarrow \lambda^2 = 3\lambda - 2 \\
\rightarrow \lambda_{1, 2} = \frac{3}{2} \pm \frac{1}{2}
$$

In generale, dato la linearità dell'equazione, la soluzione sarà quindi;

$$
s_k = A 2^k + B 1^k
$$

dove le costanti A, B vanno cercate nelle condizioni iniziali, cioè $s_0 = 0, s_1 = 1$, che portano alla soluzione analitica:

$$
s_k = 2^k - 1
$$

Per essere sicuri che questa sia la soluzione, basta inserirla nella successione definita per ricorrenza e controllare che la verifica.
In questo modo abbiamo praticamente finito, nel senso che abbiamo la chiave, che va solo tagliata a 64 bit. Notiamo inoltre che per $k \gt 64$, dato che prendiamo solo i 64 bit più bassi, la chiave è sempre $2^64 - 1$, cioè l'intero formato da 64 bit tutti settati a 1. Dato che stiamo facendo uno xor, possiamo negare tutti i bit della flag cifrata per ottenere la flag decifrata.

### Exploit

```python
# Copia dal binario
encflag = b"\x99\x93\x9e\x98\x84\x9e\x9b\xcf\x8d\xcf\x97\xa0\x93\xcb\xa0\x8d\xcc\x9c\x8a\x8d\x8c\x96\x90\x91\x9a\xa0\xcf\xcf\x9b\xc7\xca\x99\xcd\xc9\x9d\x9e\xcb\x9d\xc8\x9c\xca\x9d\xc6\x99\xc9\x9d\x9c\xcc\xcc\x9c\x9d\xc8\xce\xcd\xc8\xc6\x99\xc6\xc8\xca\x9c\xc9\x9b\x9c\xce\xcf\x82"
flag = bytes([b ^ 0xff for b in encflag])
print(flag)
```
