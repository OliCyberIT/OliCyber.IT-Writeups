# OliCyber.IT 2024 - Finale Nazionale

## [crypto] Next flag (96 risoluzioni)

Un semplice oneliner, così passi alla prossima challenge in fretta!

Author: Lorenzo Demeio <@Devrar>

## Panoramica

La challenge ci fornisce un sistema di codifica, dove ogni carattere `x` è sostituito con `x+next_prime(x)`.

## Soluzione

Per invertire la codifica possiamo creare una tabella con un carattere e la sua corrispondente codifica. Notiamo che questa codifica è unica, poiché `x+next_prime(x)` è una funzione strettamente crescente.

## Exploit

```py
import gmpy2
import json

sbox = [x+int(gmpy2.next_prime(x)) for x in range(256)]
flag = json.loads(open("next_output.txt").read())

res = ''

for el in flag:
    res += chr(sbox.index(el))

print(res)
```