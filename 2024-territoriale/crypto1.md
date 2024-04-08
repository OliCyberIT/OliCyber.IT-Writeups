# OliCyber.IT 2024 - Selezione territoriale

## [crypto] Random Cycles Baby (240 risoluzioni)

Se puoi invertirlo è un cifrario, altrimenti è una funzione di hash, giusto?

Autore: Lorenzo Demeio <@Devrar>

## Soluzione

La funzione `encrypt_or_hash` esegue una permutazione dei caratteri dell'input `w`. Più precisamente, allo step `i`, prende il carattere in posizione `i` (che chiamiamo `c_i`) e cicla tutti i caratteri dopo di esso del valore associato a `c_i` nella chiave.

$$
\begin{align*}
w &= c_1c_2c_3\ldots c_n\\
\text{step 1}\quad w_1 &= c_1c_{k_2}c_{k_3}\ldots c_{k_n}\\
\text{step 2}\quad w_2 &= c_1c_{k_2}c_{h_3}\ldots c_{h_n} \\
\vdots
\end{align*}
$$

dove $\{c_{k_2}, \ldots, c_{k_n}\}$ è una rotazione di $\{c_2, \ldots, c_n\}$ e $\{c_{h_h}, \ldots, c_{h_n}\}$ è una rotazione di $\{c_{k_2}, \ldots, c_{k_n}\}$.

Possiamo notare che a ogni step, i primi `i` caratteri rimangono uguali. Quindi, all'ultimo step, sappiamo che l'ultimo carattere è stato ruotato del valore associato al penultimo e così via. Possiamo quindi invertire ogni rotazione partendo dall'ultimo carattere e andando indietro.

```py
def decrypt(ct, key):
    pt = ct
    for i in range(n-1, 0, -1):
        k = key[pt[i-1]]
        pt = pt[:i] + spin(pt[i:], -k)
    return pt
```

Usando `-k` nella funzione `spin` otteniamo il risultato voluto (notiamo come questo sia possibile dato che nella funzione il nostro input viene preso modulo `len(w)`).

## Exploit

```py
with open("output.txt") as f:
    key = eval(f.readline().split(" = ")[1])
    enc_flag = eval(f.readline().split(" = ")[1])

n = len(enc_flag)

def spin(w, n):
    n = n % len(w)
    return w[-n:] + w[:-n]

def decrypt(ct, key):
    pt = ct
    for i in range(n-1, 0, -1):
        k = key[pt[i-1]]
        pt = pt[:i] + spin(pt[i:], -k)
    return pt

flag = decrypt(enc_flag, key)
print(flag)
```
