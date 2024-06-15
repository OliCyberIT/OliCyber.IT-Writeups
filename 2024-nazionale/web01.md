# OliCyber.IT 2024 - Finale Nazionale

## [web] Guess the flag! (92 risoluzioni)

Provaci

Site: [http://guesstheflag.challs.olicyber.it](http://guesstheflag.challs.olicyber.it)

Author: Lorenzo Leonardini <@pianka>

## Panoramica

La challenge ci permette di provare a indovinare la flag. Il controllo per la flag corretta viene eseguito lato client nel browser, ma il controllo è oscurato utilizzando JSFuck.

## Soluzione

In Chrome possiamo semplicemente aprire la console nelle Developer Tools e digitare `submit.onclick`. Questo ci stamperà la rappresentazione in stringa della funzione onsubmit, che è il codice JSFuck valutato. Firefox non fa questo, stampa solo "function", ma possiamo utilizzare uno dei numerosi strumenti online per decodificare JSFuck per recuperare il codice sorgente originale.

Il codice è semplicemente qualcosa del tipo:

```js
if (flag.value === 'flag{....}') {
	win();
} else {
	loose();
}
```
