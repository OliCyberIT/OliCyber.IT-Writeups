# OliCyber.IT 2022 - Competizione territoriale

## [crypto-1] Wordle (203 risoluzioni)

`I have no idea what wordle is, and at this point I'm too afraid to ask.`

La challenge ci dà un testo "cifrato" (in output.txt) e il sito web utilizzato per crearlo: [http://www.wordles.com/getmycrypto.aspx](http://www.wordles.com/getmycrypto.aspx)

### Soluzione

Visitando il sito web notiamo che il titolo della pagina riporta `CREATE YOUR OWN CRYPTOGRAM!`, sappiamo quindi che il nostro output è un crittogramma di qualche tipo. Leggendo dalla pagina di Wikipedia:

`A cryptogram is a type of puzzle that consists of a short piece of encrypted text. Generally the cipher used to encrypt the text is simple enough that the cryptogram can be solved by hand. Substitution ciphers where each letter is replaced by a different letter or number are frequently used.`

Abbiamo quindi buone possibilità che il cifrario sia una semplice sostituzione (questo è confermato anche dal fatto che nel testo cifrato vediamo due volte la stringa `cyvz` che, visto che conosciamo il formato della flag, sappiamo corrispondere alla sostituzione di `flag`).

Ci basta quindi utilizzare un qualsiasi servizio online in grado di risolvere crittogrammi/cifrari a sostituzione, come per esempio [https://quipqiup.com/](https://quipqiup.com/) e incollare il nostro testo cifrato per recuperare la flag.
