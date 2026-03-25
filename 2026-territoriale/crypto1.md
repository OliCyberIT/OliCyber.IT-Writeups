# OliCyber.IT 2026 - Selezione Territoriale

## [crypto] Not Morse Code (319 solves)

Ho provato a scrivere la flag in Morse Code, ma è venuta più lunga del previsto.

Author: Lorenzo Demeio <@Devrar>

## Panoramica

La challenge fa un encoding della flag scrivendo, per ogni carattere una serie o di punti `.` o di trattini `-`, in maniera alternata. Il numero di caratteri utilizzato è uguale al valore nell'encoding ASCII del carattere corrente della flag.

## Soluzione

Per ottenere la flag originale è sufficiente contare il numero di `.` o di `-` e convertirlo a carattere con la funzione `chr`. Facendo così per ogni sottosequenza, otteniamo tutti i caratteri della flag.

## Exploit

```python
with open("not_morse_code.txt") as rf:
    enc_flag = rf.read().strip()

chars = [".", "-"]
flag = ""
i = 0
j = 0
while not flag.endswith("}"):
    count = 0
    while i < len(enc_flag) and enc_flag[i] == chars[j % 2]:
        i += 1
        count += 1
    flag = flag + chr(count)
    j += 1

print(flag)
```
