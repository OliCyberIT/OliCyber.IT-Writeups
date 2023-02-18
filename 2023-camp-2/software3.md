# OliCyber.IT 2023 - Simulazione training camp 2

## [binary] implementation defined (6 risoluzioni)

La challenge prende il primo argomento da riga di comando e lo xora con un pezzo di codice nel binario, poi controlla se il risultato è uguale a una stringa che vuole.

### Soluzione

Per ottenere la flag basta comprendere che il pezzo di codice a cui punta (funzione `_init`) viene usato come chiave per xorare la flag, a quel punto basta prendere i primi 25 byte della funzione `_init` e xorarli con la variabile globale `str`, per ottenere la flag.

### Exploit

```python
#!/bin/bash

echo -e "flag{15_th15_c0d3_0r_n0t}"| ./challenge
```
