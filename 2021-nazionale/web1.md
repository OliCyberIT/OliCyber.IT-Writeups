# OliCyber.IT 2021 - Competizione nazionale

## [web-01] FrittoMisto 1 (37 risoluzioni)

La challenge presenta un sito web ancora in costruzione, con la possibilità di registrarsi e accedere a un'area riservata.

Tuttavia, al fine di registrarsi, è necessario avere un codice d'invito.

### Soluzione

Quando ci si prova a registrare, si può notare come l'errore "Codice di invito invalido!" venga mostrato prima ancora di effettuare richieste a un'API. Questo significa che c'è un qualche tipo di validazione lato client.

Questo non vuol dire che non vi sia alcun tipo di validazione lato server, ma ci dà la possibilità di reversare il controllo del codice d'invito per scoprire qual è il valore corretto.

Il sito è realizzato utilizzando [React](https://reactjs.org/), per cui il sorgente è pressocché illeggibile in quanto esportato da un [bundler](https://webpack.js.org/), fortunatamente sono abilitati i source map per fare debug. Ciò significa che dalla scheda "Sources" dei Dev Tools del browser è possibile navigare i file `.js` originali.

Da qua possiamo leggere il controllo del codice d'invito:

```js
if (inviteCode.length !== 10) {
  console.log("Codice di invito invalido!");
  props.setError("Codice di invito invalido!");
  return;
}
for (let idx = 0; idx < 10; idx++) {
  if (inviteCode.charCodeAt(idx) != idx) {
    console.log("Codice di invito invalido!");
    props.setError("Codice di invito invalido!");
    return;
  }
}
```

Il codice corretto è quindi lungo 10 caratteri ed è composto dai byte `0x0`, `0x1`, ..., `0x9`

### Exploit

Il modo più semplice per registrarsi è effettuare una richiesta all'API utilizzando Python:

```python
import requests

r = requests.post("http://frittomisto.challs.olicyber.it/api/register", json={
    "username": "myusername",
    "password": "mypassword",
    "invite": "\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09"
})
print(r.text)
```
