# OliCyber.IT 2022 - Competizione territoriale

## [misc-1] Do you believe in magic? (155 risoluzioni)

`I do believe in magic! Now tell me what's going on with this file!`

Viene dato un file i cui primi byte sono stati cambiati, lo scopo è ripristinarli e visualizzare l'immagine png.

### Soluzione

Con `xxd` si vede che i primi 8 bytes sono stati sostituiti con: "BA AA AA AA AA AA AA AD". Per ripristinare il file basta sostituirli con l'header standard dei file png: "89 50 4E 47 0D 0A 1A 0A" (https://en.wikipedia.org/wiki/List_of_file_signatures). Si può fare con un hex editor o semplicemente con bash.

### Exploit

`(echo -ne "\x89\x50\x4E\x47\x0D\x0A\x1A\x0A"; tail -c +9 chall.png) > flag.png`

![flag](attachments/flag.png)
