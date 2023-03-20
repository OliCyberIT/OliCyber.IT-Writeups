# OliCyber.IT 2023 - Selezione territoriale

## [crypto-1] Classicamente python (381 risoluzioni)

La challenge cifra la flag con un columnar transposition cipher.

### Soluzione

Possiamo notare che la flag viene paddata fino ad avere lunghezza multipla di 4, e poi si creano quattro blocchi.
Ogni blocco contiene i caratteri in posizione i, i+4, i+8, etc. Questo corrisponde a scrivere la flag in una tabella con 4 colonne scrivendola riga per riga, e poi leggendola per colonne.

Nel testo cifrato, la prima colonna corrisponde al primo quarto di testo `enc[:len(enc)//4]` e così via.
Per ottenere la flag basta dunque prendere i quattro pezzi del testo cifrato, metterli in una "tabella" come colonne, e poi leggere riga per riga.

### Exploit

```python
enc = "f{anuiraaso_lfltnfi_sin_aime_rotpze_gne_ca_roi}_"

n = len(enc)//4
cols = [enc[n*i:n*(i+1)] for i in range(4)]

flag = ""
for i in range(n):
    flag += "".join(col[i] for col in cols)
print(flag)
```
