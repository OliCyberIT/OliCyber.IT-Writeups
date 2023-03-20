# OliCyber.IT 2023 - Selezione territoriale

## [binary-2] Super-licenza (89 risoluzioni)

Abbiamo un altro binario risultato della compilazione di un file C. Il programma ci offre la possibilità di validare una super-licenza.
Ci dice sia se la stringa che gli abbiamo data è corretta, sia se è sbagliata. Per capire qualcosa del binario è necessario analizzarlo.

### Soluzione

Anche se il binario è strippato, possiamo comunque analizzarlo per capire cosa fa, per esempio caricandolo su ghidra.
Uno dei primi controlli è la lunghezza della stringa che gli mandiamo, che deve essere uguale a 52 caratteri.

La funzione successiva, fa un pattern che assomiglia a

```c
buf2[i] = buf1[array[i]];
```

che sembra un modo per permutare o quantomeno cambiare l'ordine degli elementi di `buf1`. Possiamo con un banale copia-incolla prenderci questo array e metterlo da parte.

La funzione successiva, dopo un breve rename delle variabili, fa una cosa molto semplice: xora il risultato della funzione di prima con una chiave hard-codata nel binario. Possiamo di nuovo con un copia-incolla salvarci questa chiave.

Dopo aver fatto queste due modifiche alla nostra stringa, il programma la confronta con un'altra stringa hardcodata nel binario, che possiamo salvarci. A questo punto, se vogliamo ottenere la flag, possiamo prendere questa stringa hardcodata e applicare all'inverso le operazioni che fa il binario.

L'inverso dello xor con una chiave fissa è semplicemente fare di nuovo lo xor con la stessa chiave. La permutazione è leggermente più tricky, perché dobbiamo ottenere l'inverso di una permutazione, cosa che si può fare in modo semplice con python come indicato sotto.
Mettendo insieme tutti i pezzi si ottiene la flag.

### Exploit

```python
perm = [13, 25, 31, 10, 11, 15, 44, 51, 4, 46, 19, 28, 22, 50, 9, 30, 18, 20, 0, 26, 45, 42, 6, 48, 2, 39, 16, 7, 8, 24, 34, 17, 37, 36, 14, 3, 41, 33, 12, 23, 1, 40, 35, 49, 27, 21, 29, 43, 32, 47, 5, 38]
key = [154, 248, 31, 43, 27, 224, 171, 31, 195, 98, 254, 218, 168, 63, 112, 60, 117, 25, 48, 160, 72, 193, 84, 202, 117, 230, 117, 166, 222, 22, 110, 239, 24, 237, 230, 252, 228, 17, 6, 163, 175, 94, 29, 36, 246, 93, 202, 142, 163, 234, 150, 165]
enc_flag = [170, 167, 125, 116, 105, 129, 146, 98, 184, 7, 207, 168, 156, 7, 17, 99, 23, 119, 86, 216, 121, 241, 33, 172, 20, 130, 42, 150, 170, 115, 89, 156, 41, 218, 146, 155, 208, 112, 115, 252, 195, 63, 120, 64, 198, 51, 254, 239, 149, 222, 228, 199]

dec_flag = [k ^ f for k, f in zip(key, enc_flag)]
inv_perm = [perm.index(v) for v in range(len(perm))]
flag = bytes([dec_flag[v] for v in inv_perm]).decode()
print(flag)
```
