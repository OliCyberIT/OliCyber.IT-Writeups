# OliCyber.IT 2025 - Selezione territoriale

## [web] Segnalazione cinghiali cittadini (128 risoluzioni)

Finalmente in comune hanno sviluppato una piattaforma per raccogliere segnalazioni sui cinghiali trovati in giro.

Ho come la sensazione che nessuno approvi queste segnalazioni...

Site: [http://cinghiali.challs.territoriali.olicyber.it](http://cinghiali.challs.territoriali.olicyber.it)

Autore: Lorenzo Leonardini <@pianka>

## Panoramica

Il sito si presenta in maniera molto semplice: dopo una rapida registrazione possiamo visualizzare la lista delle nostre segnalazioni inviate, nonché creare una nuova segnalazione.

Leggendo il codice sorgente, notiamo che non appena creiamo una segnalazione questa viene immediatamente visualizzata dall'admin.

## Soluzione

Sebbene ci sia un raffazzonato tentativo di prevenire XSS eliminando i caratteri `<` e `>`, possiamo iniettare un event handler nel tag `<img>`. Il template engine (ejs), infatti, non viene utilizzato correttamente e non sanitizza le XSS.

## Exploit

Possiamo creare una segnalazione che come immagine ha

```js
x" onerror="document.forms[0].submit()
```

per inviare automaticamente il form di approvazione.
