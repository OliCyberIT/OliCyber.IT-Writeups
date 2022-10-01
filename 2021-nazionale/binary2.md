# OliCyber.IT 2021 - Competizione nazionale

## [binary-2] Algoritmo Sicuro e Resistente (19 risoluzioni)

La challenge è un binario strippato che decifra la flag quattro caratteri alla volta e li compara con l'input dell'utente.

### Soluzione

Decompilando il binario notiamo che il risultato della chiamata a `strlen()` deve essere uguale a 21:

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  const char *v3; // rbx
  const char *v4; // rax
  char s[8]; // [rsp+20h] [rbp-40h] BYREF
  __int64 v7; // [rsp+28h] [rbp-38h]
  __int64 v8; // [rsp+30h] [rbp-30h]
  __int64 v9; // [rsp+38h] [rbp-28h]
  unsigned int i; // [rsp+4Ch] [rbp-14h]

  *(_QWORD *)s = 0LL;
  v7 = 0LL;
  v8 = 0LL;
  v9 = 0LL;
  puts("Dammi la flag!");
  fgets(s, 32, stdin);
  s[strcspn(s, "\n")] = 0;
  if ( strlen(s) != 21 )
  {
    puts(aQuellaNon);
    exit(1);
  }
  puts("Sto analizzando il tuo input...");
  for ( i = 0; (int)i <= 5; ++i )
  {
    v3 = &s[4 * i];
    v4 = (const char *)decrypt(i);
    if ( strncmp(v4, v3, 4uLL) )
    {
      puts(aQuellaNon);
      exit(2);
    }
  }
  puts(aSiQuella);
  return 0;
}
```

Ora che sappiamo la lunghezza della flag possiamo usare `ltrace` per controllare le varie chiamate che verranno fatte alla `strncmp` e ottenere la flag.

```sh
$ echo "AAAAAAAAAAAAAAAAAAAAA" | ltrace -e strncmp ./ASR >/dev/null
ASR->strncmp("flag", "AAAAAAAAAAAAAAAAAAAAA", 4)                                                 = 37
+++ exited (status 2) +++

$ echo "flagAAAAAAAAAAAAAAAAA" | ltrace -e strncmp ./ASR >/dev/null
ASR->strncmp("flag", "flagAAAAAAAAAAAAAAAAA", 4)                                                 = 0
ASR->strncmp("{b4d", "AAAAAAAAAAAAAAAAA", 4)                                                     = 58
+++ exited (status 2) +++

$ echo "flag{b4dAAAAAAAAAAAAA" | ltrace -e strncmp ./ASR >/dev/null
ASR->strncmp("flag", "flag{b4dAAAAAAAAAAAAA", 4)                                                 = 0
ASR->strncmp("{b4d", "{b4dAAAAAAAAAAAAA", 4)                                                     = 0
ASR->strncmp("_pr0", "AAAAAAAAAAAAA", 4)                                                         = 30
+++ exited (status 2) +++

$ echo "flag{b4d_pr0AAAAAAAAA" | ltrace -e strncmp ./ASR >/dev/null
ASR->strncmp("flag", "flag{b4d_pr0AAAAAAAAA", 4)                                                 = 0
ASR->strncmp("{b4d", "{b4d_pr0AAAAAAAAA", 4)                                                     = 0
ASR->strncmp("_pr0", "_pr0AAAAAAAAA", 4)                                                         = 0
ASR->strncmp("gr4m", "AAAAAAAAA", 4)                                                             = 38
+++ exited (status 2) +++

$ echo "flag{b4d_pr0gr4mAAAAA" | ltrace -e strncmp ./ASR >/dev/null
ASR->strncmp("flag", "flag{b4d_pr0gr4mAAAAA", 4)                                                 = 0
ASR->strncmp("{b4d", "{b4d_pr0gr4mAAAAA", 4)                                                     = 0
ASR->strncmp("_pr0", "_pr0gr4mAAAAA", 4)                                                         = 0
ASR->strncmp("gr4m", "gr4mAAAAA", 4)                                                             = 0
ASR->strncmp("m1ng", "AAAAA", 4)                                                                 = 44
+++ exited (status 2) +++

$ echo "flag{b4d_pr0gr4mm1ngA" | ltrace -e strncmp ./ASR >/dev/null
ASR->strncmp("flag", "flag{b4d_pr0gr4mm1ngA", 4)                                                 = 0
ASR->strncmp("{b4d", "{b4d_pr0gr4mm1ngA", 4)                                                     = 0
ASR->strncmp("_pr0", "_pr0gr4mm1ngA", 4)                                                         = 0
ASR->strncmp("gr4m", "gr4mm1ngA", 4)                                                             = 0
ASR->strncmp("m1ng", "m1ngA", 4)                                                                 = 0
ASR->strncmp("}", "A", 4)                                                                        = 60
+++ exited (status 2) +++

$ echo "flag{b4d_pr0gr4mm1ng}" | ./ASR
Dammi la flag!
Sto analizzando il tuo input...
Si! Quella è la flag!
```
