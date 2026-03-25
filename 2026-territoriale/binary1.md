# OliCyber.IT 2026 - Selezione Territoriale

## [binary] Trace the void (220 solves)

Non ti consiglio di reversare questo programma. Però con qualche tool di analisi dinamica... maybe you can **trace** the void!

Author: Alberto Carboneri <@Alberto247>

## Panoramica

La challenge è incentrata sull'applicazione di tecniche di analisi dinamica per estrarre informazioni nascoste dall'esecuzione di un binario. A differenza del reverse engineering statico, dove il codice viene decompilato e studiato visivamente, la descrizione suggerisce che la soluzione ottimale si trova analizzando il programma mentre esegue.

## Soluzione

Utilizzando uno strumento come `ltrace`, è possibile intercettare e registrare tutte le chiamate dinamiche a librerie condivise eseguite dal binario.
In fase di esecuzione il programma invoca ripetutamente la funzione `putchar()` per stampare a schermo la flag nascosta carattere per carattere.

Basta eseguire `ltrace` con il flag `-e putchar` per estrapolare direttamente questo specifico comportamento, ricostruendo l'intera stringa.

## Exploit

```python
import subprocess
import re

result = subprocess.run(
    ["ltrace", "-e", "putchar", "./void"],
    capture_output=True,
    text=True
)
chars = re.findall(r"putchar\((\d+)[^\d]", result.stderr)
print("".join(chr(int(c)) for c in chars))
```
