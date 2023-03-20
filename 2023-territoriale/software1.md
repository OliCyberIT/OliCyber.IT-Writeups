# OliCyber.IT 2023 - Selezione territoriale

## [binary-1] Frasi basate (336 risoluzioni)

La challenge è un semplice esegubile ELF molto piccolo, risultato della compilazione di un programma in C. La descrizione della challenge dice che ci sono delle informazioni da recuperare da questo binario. Non essendoci un server remoto indicato, la challenge è statica e molto probabilmente di puro reverse.

### Soluzione

Usando `strings` sul binario notiamo che ci sono alcune stringhe evidentemente stampabili, e una particolarmente lunga

```bash
Qm9pYSBkZWgsIGxhIGZsYWcgaW4gYjY0LCBpbnRyb3ZhYmlsZSwgYXNzb2x1dGFtZW50ZSBmbGFne3N0cjFuZ2gzX2ludXQxbDFfZV9kMHYzX3RyMHY0cmwzXzNiM2RhNDYzY2NhZTNhOTVmODMxfQ==
```

Dato che finisce per `==`, sospettiamo sia il base64 di una stringa. Possiamo decodificarla

```bash
echo -n "Qm9pYSBkZWgsIGxhIGZsYWcgaW4gYjY0LCBpbnRyb3ZhYmlsZSwgYXNzb2x1dGFtZW50ZSBmbGFne3N0cjFuZ2gzX2ludXQxbDFfZV9kMHYzX3RyMHY0cmwzXzNiM2RhNDYzY2NhZTNhOTVmODMxfQ==" | base64 -d

Boia deh, la flag in b64, introvabile, assolutamente flag{str1ngh3_inut1l1_e_d0v3_tr0v4rl3_3b3da463ccae3a95f831}
```

e trovare effettivamente la flag.

### Exploit

```bash
#!/usr/bin/env bash
strings -n 80 based | base64 -d

```
