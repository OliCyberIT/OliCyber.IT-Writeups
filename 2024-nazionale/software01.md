# OliCyber.IT 2024 - Finale Nazionale

## [rev] MIC (31 risoluzioni)

É arrivato un nuovo Messaggio In Chiavetta per lei Agente.

Author: Alberto Carboneri <@Alberto247>

## Panoramica

La challenge è composta da un singolo binario che chiede all'utente un input.
L'input viene quindi elaborato e verificato per essere uguale a una stringa hardcoded.
Se il controllo viene superato, l'input dell'utente viene poi xorato con un valore statico in memoria, producendo la flag.

## Soluzione

La parte principale della challenge consiste nel reverse engineering della funzione `check_key` per ottenere l'input corretto dall'output disponibile.
Ci sono due modi per raggiungere questo obiettivo: analisi statica e dinamica.
Quest'ultima è la più semplice, poiché la funzione `check_key` rimescola solo l'input senza sostituire alcuna lettera. Possiamo quindi fornire come input una stringa composta da caratteri unici, come "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789", analizzare la memoria nella funzione `strncmp` e produrre una mappa che associa ogni carattere di input alla sua posizione di output. Poi, invertendo questa mappa e applicandola alla variabile codificata `encrypted_key`, possiamo ottenere l'input corretto.

L'approccio statico, sebbene più complicato, può essere risolto deducendo il funzionamento interno delle funzioni dai loro nomi, con l'aiuto di un decompilatore come Ghidra.
- `format_key` formatta il nostro input in una matrice quadrata di dimensione 6x6 (la variabile `grille`).
- `rotate_grille` ruota la variabile `grille` di 90 gradi.
- `check_key` estrae caratteri specifici dalla `grille`, quindi la ruota, per 4 volte.

Invertendo il processo possiamo recuperare l'input dall'output fornito.

## Exploit

```python
from pwn import *

chall = ELF("./chall")

encrypted_key=chall.read(chall.symbols["encrypted_key"], 36).decode()
encrypted_flag=chall.read(chall.symbols["flag"], 36)

holes=[[0,1],[1,1],[1,3],[3,1],[3,3],[4,5],[5,0],[5,2],[5,3]] # Obtained from the binary

def grille_encrypt(data):
    matrix=[[0,0,0,0,0,0],[0,0,0,0,0,0],[0,0,0,0,0,0],[0,0,0,0,0,0],[0,0,0,0,0,0],[0,0,0,0,0,0]]
    ctr=0
    for x in range(4):
        for y in range(9):
            matrix[holes[y][0]][holes[y][1]]=data[ctr]
            ctr+=1
        matrix = [[matrix[j][i] for j in range(len(matrix))] for i in range(len(matrix[0])-1,-1,-1)]
        matrix = [[matrix[j][i] for j in range(len(matrix))] for i in range(len(matrix[0])-1,-1,-1)]
        matrix = [[matrix[j][i] for j in range(len(matrix))] for i in range(len(matrix[0])-1,-1,-1)]
    return "".join(["".join(x) for x in matrix])

key = grille_encrypt(encrypted_key)
print(xor(encrypted_flag, key.encode()).decode())
```
