# OliCyber.IT 2023 - Simulazione training camp 3

## [binary] lil-overflow (41 risoluzioni)

La challenge chiede di inserire una stringa, e la scrive in un buffer sullo stack.

Il numero di byte che il programma legge è maggiore della lunghezza del buffer, quindi si può sovrascrivere la memoria che sullo stack si trovo dopo il buffer.

Il programma poi confronta una variabile sullo stack con _0x5ab1bb0_, e se corrispondesse, stamperebbe la flag.

### Soluzione

Dato che il buffer ha dimensione 32, e subito dopo sullo stack c'è un puntatore non utilizzato e la variabile di interesse, si può sovrasrcrivere quest'ultima scrivendo 40 byte seguiti dal valore che ci interessa scrivere.

### Exploit

```sh
echo -e "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\xb0\x1b\xab\x05"| nc IP PORT
```
