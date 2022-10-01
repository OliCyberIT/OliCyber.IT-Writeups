# OliCyber.IT 2021 - Competizione nazionale

## [binary-1] La settimana enigmistica (23 risoluzioni)

La flag viene checkata utilizzandola come input di un sudoku

Board utilizzata:

```py
sudokuMatrix = [
    [5, 0, 0, 0, 0, 3, 0, 8, 0],
    [0, 0, 3, 0, 4, 0, 0, 0, 0],
    [7, 8, 0, 0, 0, 0, 0, 9, 0],
    [0, 0, 1, 2, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 1, 9, 6, 0],
    [0, 0, 7, 0, 0, 0, 0, 3, 2],
    [0, 0, 0, 4, 0, 0, 5, 0, 0],
    [0, 0, 2, 0, 0, 9, 0, 0, 0],
    [0, 9, 0, 1, 2, 0, 0, 0, 6],
]
```

Nel binario la board viene salvata sottoforma di stringa:  
`500003080003040000780000090001200000000001960007000032000400500002009000090120006`  
In `board_ctor` viene trasformata in numeri nel range [0-9]

Output saliente di `strings`:

```
flag{
Voglio una flag piu' enigmistica! :(
Complimenti! Il check enigmistico e' andato a buon fine :)
Nop :(
500003080003040000780000090001200000000001960007000032000400500002009000090120006
check_rows
check_columns
check_subsquare
check_all_subsquares
```

### Soluzione

Dai nomi delle funzioni e con l'aiuto delle parole enigmistico/a,
è molto facile capire cosa sta facendo il binario.

La soluzione è semplicemente risolvere quel sudoku 9x9, spero che non ci siano persone
che lo faranno a mano.

### Exploit

```py
# Spazi riempiti con la flag
idxs = [ 1, 2, 3, 4, 6, 8, 9, 10, 12, 14, 15, 16, 17, 20, 21, 22, 23, 24, 26, 27, 28, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 44, 45, 46, 48, 49, 50, 51, 54, 55, 56, 58, 59, 61, 62, 63, 64, 66, 67, 69, 70, 71, 72, 74, 77, 78, 79 ]
# Board solvata
solution = [
    5, 4, 9, 7, 6, 3, 2, 8, 1,
    2, 1, 3, 9, 4, 8, 6, 5, 7,
    7, 8, 6, 5, 1, 2, 3, 9, 4,
    6, 3, 1, 2, 9, 5, 4, 7, 8,
    8, 2, 4, 3, 7, 1, 9, 6, 5,
    9, 5, 7, 6, 8, 4, 1, 3, 2,
    1, 7, 8, 4, 3, 6, 5, 2, 9,
    4, 6, 2, 8, 5, 9, 7, 1, 3,
    3, 9, 5, 1, 2, 7, 8, 4, 6,
]

print ('flag{', end='')
for i in idxs:
    print (solution[i], end='')
print ('}')
# flag{497621219865765123463954788243759568411783629468571335784}
```
