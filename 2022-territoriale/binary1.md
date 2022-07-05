# OliCyber.IT 2022 - Competizione territoriale

## [binary-1] Secure Gate (196 risoluzioni)

La challenge è una finta shell che permette di eseguire i comandi ls e cat. ls ritorna solo la stringa "flag.txt" e "cat flag.txt" richiede una password per essere usato.

### Soluzione

Osservando il decompilato usando un tool come Ghidra si nota il seguente pezzo di codice nella funzione checkPassword:

```C
if(strncmp(password, "notmysecurepassword", 19)==0){
    return 1;
}
```

Inserendo come password "notmysecurepassword" si ottiene la flag.

### Exploit

```bash
printf "cat flag.txt\nnotmysecurepassword\n" | ./secureGate
```
