# OliCyber.IT 2023 - Simulazione training camp 1

## [web] Gabibbo Says (195 risoluzioni)

Ci viene presentata una pagina web con l'indicazione di eseguire una richiesta HTTP.

### Soluzione

Per risolvere la challenge è sufficiente effettuare una richiesta POST, usando ad esempio il modulo `requests` di python, inviando come dati della richiesta quelli indicati dalla consegna della challenge.

La flag sarà quindi contenuta nel testo di risposta alla richiesta.

### Exploit

```python
#!/bin/python3

import re
import os
import requests
import logging
logging.disable()


# Per le challenge web
URL = os.environ.get("URL", "http://gabibbo-says.challs.olicyber.it")

flag_regex = "flag{.*}"

r = requests.post(URL, data={
    "gabibbo": "angry"
})

flag = re.findall(flag_regex, r.text)[0]

print(flag)

```
