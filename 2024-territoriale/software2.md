# OliCyber.IT 2024 - Selezione territoriale

## [binary] Ghost (85 risoluzioni)

Questo binario fa paura, ha dei fantasmi dentro.

Autore: Fabio Zoratti <@orsobruno96>

## Panoramica

Il binario fornito è un programma C compilato. Dopo l'ingresso nella funzione `main`, il programma chiama altre funzioni sconosciute che eseguono operazioni su due buffer globali che sono inizialmente pieni di dati apparentemente casuali,

```as
│           0x004014b2      55             push rbp
│           0x004014b3      4889e5         mov rbp, rsp
│           0x004014b6      ba00010000     mov edx, 0x100              ; 256 ; uint32_t arg3
│           0x004014bb      be00424000     mov esi, 0x404200           ; uint32_t arg2
```

dopodichè stampa in output dati che sembrano irrilevanti, ed esce.

## Soluzione

L'analisi statica è possibile: si può entrare in profondità nel reverse engineering per comprendere le operazioni svolte nelle funzioni sconosciute. Tuttavia, c'è un modo molto più conveniente: usare l'analisi dinamica. Possiamo usare `gdb` per impostare un breakpoint subito dopo l'esecuzione di queste funzioni, subito prima della stampa finale, ed esaminare il contenuto dei buffer globali per ottenere la flag.

```text
gdb> break printf
run
x/s 0x404200
flag{gdb_1s_4ctuall1_us3ful_78ca281c}
```
