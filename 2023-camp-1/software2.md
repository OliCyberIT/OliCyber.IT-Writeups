# OliCyber.IT 2023 - Simulazione training camp 1

## [binary] flagvault (40 risoluzioni)

La challenge compone una stringa random con la funzione getPass() e la chiede all'utente.

### Soluzione

Il PRNG è unseeded, quindi la password composta è facilmente prevedibile.

Con un debugger si può impostare un breakpoint alla funzione `strncmp` che controlla se la password è giusta per leggerla dalla memoria.

### Exploit

```sh
echo 49CMO:N?O2CD | nc $IP $PORT
```
