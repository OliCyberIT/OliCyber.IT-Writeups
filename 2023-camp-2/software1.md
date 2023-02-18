# OliCyber.IT 2023 - Simulazione training camp 2

## [binary] strong-as-md5 (20 risoluzioni)

La challenge controlla se la stringa inserita corrisponda alla flag, attraverso delle operazioni eseguite sull'input.

In particolare aggiunge ad ognun carattere dell'input inserito 4 valori, diversi per ogni char, definiti in una costante globale, e poi controlla che l'output finale corrisponda ad un target, sempre definito globalmente.

### Soluzione

Dato che ogni operazione su ogni singolo carattere è fissa, basta invertirla per ottenere la flag originale dal target definito nel programma.

### Exploit

```c
#include <stdio.h>
#include <stdlib.h>

char table[64] = "\xad\xf7\xb1\x83`S\xb7^D\x91UK\xbc\xfb\x85[?\xecU\xd5!\xe3\xb1\xa9\x15\xc9\x90H\xc2%.0\x06\xb3\xa6\xe4\xf5\xac~\x12\xadk\xc2\x82\xbf\r\xda8a*\x0fz\xdd\x8f\x9a\xaeh\x14\xa4\xa6\xd5\xd7 \x96";
char input[16] = "\x67\x7c\xea\x32\x8b\xc0\xc6\x9f\xda\xd7\xe8\x1c\xd3\xf6\xaf\xb5";

int main() {
  for (int i = 0; i < 4; i++) {
    for (int j = 0; j < 16; j++) {
      input[j] = input[j] - table[i * 16 + j];
    }
  }
  puts(input);
}
```
