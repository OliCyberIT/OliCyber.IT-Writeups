# OliCyber.IT 2022 - Competizione nazionale

## [web-3] Bibvault 1 (9 risoluzioni)

Il sito permette di registrarsi ed eseguire il login, ogni volta che viene commessa una delle due azioni, viene impostato un token jwt, il token viene successivamente controllato da questa funzione

```js
const authenticateToken = (req, res, next) => {
  let token = null;
  let cookies = req.headers.cookie;
  if (cookies) {
    cookies = cookies
      .split(";")
      .find((e) => e.split("=")[0].replace(" ", "") === "auth");
    if (cookies != undefined) token = cookies.split("=")[1].replace(" ", "");
  }

  if (token == null) {
    req.user = null;
    next();
    return;
  }
  const user = jwt.decode(token);
  if (typeof user.username !== "string") return res.sendStatus(400);
  req.user = user;
  next();
};
```

### Soluzione

La funzione `authenticateToken` utilizza la libreria jsonwebtokens, in particolare utilizza la funzione `jwt.decode`, sulla documentazione della libreria si può scoprire che questa funzione non fa un controllo sulla firma, di conseguenza si può impostare lo username _"gabibbo"_ all'interno del jwt e prendere la flag salvata nella sua home.

```json
{
  "username": "gabibbo",
  "is_admin": true
}
```
