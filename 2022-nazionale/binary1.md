# OliCyber.IT 2022 - Competizione nazionale

## [binary-1] Dati Privati (7 risoluzioni)

La challengepermette all'utente di inserire del codice `cpp`, che verrà compilato ed eseguito dal server remoto.
In allegato si trova il file .cpp che viene utilizzato come template dal servizio remoto.

```cpp
#include <vector>

class Flag {
    std::vector<char> *flag;
public:
    Flag() {
        flag = new std::vector<char>({102,108,97,103,123,81,85,69,83,84,65,95,70,76,65,71,95,69,39,95,83,73,67,85,82,65,77,69,78,84,69,95,85,78,65,95,70,76,65,71,95,86,69,82,65,125});
    }
};

// Il tuo codice va qua
```

### Soluzione

Lo scopo della challenge è quello di accedere ad un field privato di una classe cpp.

Il linguaggio non permette l'accesso diretto ai field privati, quindi una cosa del tipo

```cpp
    Flag A;
    A.flag;
```

risulterebbe in un errore.

Questo limite ovviamente non è security boundary, possiamo bypassarlo giocando con i puntatori.

Compilando un programma d'esempio con le informazioni di debug, possiamo ispezionare da gdb il layout di questa classe in memoria:

```
     10
     11	 // Il tuo codice va qua
     12	 int main()
     13	 {
     14	     Flag A;
 →   15	 }
──────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "a.out", stopped 0x5555555552d0 in main (), reason: SINGLE STEP
────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x5555555552d0 → main()
─────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤  telescope &A
0x007fffffffdcd0│+0x0000: 0x0055555556aeb0  →  0x0055555556aed0  →  "flag{QUESTA_FLAG_E'_SICURAMENTE_UNA_FLAG_VERA}"	 ← $rsp
0x007fffffffdcd8│+0x0008: 0xbab8ed47f42dab00
0x007fffffffdce0│+0x0010: 0x0000000000000001	 ← $rbp
0x007fffffffdce8│+0x0018: 0x007ffff7b5ad90  →  <__libc_start_call_main+128> mov edi, eax
```

Cosa sta facendo il comando telescope di gdb? Sta semplicemente dereferenziando puntatori fino a che può

- `&A` = 0x007fffffffdcd0
- `*&A` = 0x0055555556aeb0
- `**&A` = 0x0055555556aed0
- `puts(**&A)` -> "flag{QUESTA_FLAG_E'\_SICURAMENTE_UNA_FLAG_VERA}"

Quindi ci basta creare un programma equivalente in cpp che dereferenzia i puntatori in modo analogo a telescope.

```cpp
#include <stdio.h>

int main(int argc, char **argv)
{
    Flag A;

    puts(**(char***)&A);
    return 0;
}
```

Altra soluzione equivalente la possiamo ottenere utilizzando i costrutti del cpp.

```cpp
#include <stdio.h>

int main(int argc, char **argv)
{
    Flag A;
    std::vector<char> *flag = *(std::vector<char> **)&A;
    for(auto ch : *flag)
        printf("%c", ch);
    puts("");
}
```

### Exploit

Inviando uno dei due codici al server otteniamo la flag:

```
nc privatedata.challs.olicyber.it 12300
Inserisci del codice c++ seguito da una riga che inizia per 'EOF'
Ad esempio:

#include <stdio.h>
int main()
{
    puts("hello world");
    return 0;
}
EOF

#include <stdio.h>

int main(int argc, char **argv)
{
    Flag A;

    puts(**(char***)&A);
    return 0;
}
EOF
b"flag{in_nome_dell'undefined_behaviour_ti_dono_questa_flag}\n"
```
