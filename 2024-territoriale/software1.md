# OliCyber.IT 2024 - Selezione territoriale

## [binary] Section31 (396 risoluzioni)

Per risolvere questa challenge devi guardare l'intera serie Star Trek. GLHF

Autore: Fabio Zoratti <@orsobruno96>

## Soluzione

Il file è un piccolo ELF stripped che sembra non fare niente a parte stampare qualche cosa. Il nome della challenge ci suggerisce che sia una buona idea dare un'occhiata alle sezioni del file ELF. Utilizzando `readelf -a` notiamo che ci sono diverse sezioni in questo file con un nome sospetto:

```text
  [24] .data             PROGBITS         0000000000404008  00003008
       0000000000000004  0000000000000000  WA       0     0     1
  [25] .bss              NOBITS           000000000040400c  0000300c
       0000000000000004  0000000000000000  WA       0     0     1
  [26] .comment          PROGBITS         0000000000000000  0000300c
       000000000000002e  0000000000000001  MS       0     0     1
  [27] .annobin.notes    STRTAB           0000000000000000  0000303a
       000000000000018c  0000000000000001  MS       0     0     1
  [28] flag_0_f          PROGBITS         0000000000000000  000031c6
       0000000000000001  0000000000000000           0     0     1
  [29] flag_1_l          PROGBITS         0000000000000000  000031c7
       0000000000000001  0000000000000000           0     0     1
  [30] flag_2_a          PROGBITS         0000000000000000  000031c8
       0000000000000001  0000000000000000           0     0     1
  [31] flag_3_g          PROGBITS         0000000000000000  000031c9
       0000000000000001  0000000000000000           0     0     1
  [32] flag_4_{          PROGBITS         0000000000000000  000031ca
       0000000000000001  0000000000000000           0     0     1
  [33] flag_5_f          PROGBITS         0000000000000000  000031cb
       0000000000000001  0000000000000000           0     0     1
```

Prendendo tutte le sezioni con nome `flag_[0-9]+_[a-z]` e concatenando l'ultima lettera di ciascuna di esse, otteniamo la flag. Questo può essere fatto a mano oppure con un oneliner:

```bash
readelf -a $1 | grep flag_ | cut -d '_' -f3 | cut -d ' ' -f1 | awk '{print}' ORS=""
```
