# OliCyber.IT 2025 - Selezione territoriale

## [crypto] Insecure Passphrase Generator (522 risoluzioni)

Eccovi uno script per generare una passphrase lunghissima, però non la userei per il vostro password manager forse...

Autore: Lorenzo Demeio <@Devrar>

## Panoramica

La challenge crea una passphrase a partire dalla flag, sostituendo ogni lettera minuscola con una parola e lasciando gli altri caratteri invariati.

## Soluzione

Notiamo che la parola associata ad ogni lettera è univoca ed è determinata della posizione della lettera nell'alfabeto. Quindi la `a` viene sostituita dalla prima parola nella lista, ovvero `casa`, la `b` dalla seconda, ovvero `albero`, e così via.

Per tornare indietro è dunque sufficiente determinare l'indice di ogni parola all'interno della lista e sostituirlo con la lettera alla posizione corrispondente nell'alfabeto.

## Exploit

```py
words = [
    "casa", "albero", "notte", "sole", "montagna", "fiume", "mare", "vento", "nuvola", 
    "pioggia", "strada", "amico", "sorriso", "viaggio", "tempo", "cuore", "stella", 
    "sogno", "giorno", "libro", "porta", "luce", "ombra", "silenzio", "fiore", "luna"
]

with open("passphrase.txt", "r") as rf:
    enc_flag = rf.read()

flag = ""
for w in enc_flag.split('-'):
    if w in words:
        flag += chr(words.index(w) + ord('a'))
    else:
        flag += w

print(flag)
```