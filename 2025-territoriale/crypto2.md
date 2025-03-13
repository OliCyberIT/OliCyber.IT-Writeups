# OliCyber.IT 2025 - Selezione territoriale

## [crypto] Again? (156 risoluzioni)

Alla fine è sempre il solito xor...

_Nota: la stringa `text` contiene solo caratteri stampabili_

Autore: Matteo Rossi <@mr96>

## Panoramica

La cifratura del messaggio avviene in due parti:

- una sostituzione byte per byte con una SBOX fissa e nota
- lo xor con una chiave di lunghezza casuale tra 6 e 12 e ripetuta fino a coprire il testo

Il testo cifrato è molto grande e la flag viene concatenata alla fine, ma togliendo il flag format.

## Soluzione

La lunghezza del plaintext suggerisce che sia possibile effettuare un attacco ad analisi di frequenza. Infatti, l'applicazione della SBOX corrisponde ad una sostituzione monoalfabetica sull'insieme dei bytes e mantiene quindi la frequenza dei caratteri. Lo xor con la chiave ripetuta è invece analogo ad una cifratura con Vigenere e può essere portato lo stesso tipo di attacco.

Fare lo xor con un byte corrisponde a fare una permutazione dei bytes e mantiene quindi le frequenze. Tuttavia la chiave non è di un solo byte, ma di un numero casuale tra 6 e 12.

Chiamando `l` la lunghezza della chiave, abbiamo però che tutti i caratteri a distanza `l` sono xorati con lo stesso byte della chiave ed è quindi possible eseguire un attacco ad analisi di frequenza su di essi.

Un attacco simile è eseguibile manualmente o tramite dei tool automatici come [xortool](https://github.com/hellman/xortool). Provando tutti i possibili valori di `l` ed applicando la SBOX inversa, possiamo verificare di aver trovato la chiave giusta se tutti i caratteri che otteniamo sono stampabili.

## Exploit

```py
import os
import string
import subprocess
import sys

SBOX = [23, 46, 93, 178, 209, 80, 169, 227, 246, 14, 79, 139, 196, 109, 176, 76, 188, 74, 163, 187, 130, 110, 101, 241, 202, 239, 53, 117, 114, 72, 131, 217, 71, 55, 253, 45, 212, 191, 59, 30, 104, 190, 251, 20, 94, 211, 84, 85, 68, 73, 237, 205, 174, 97, 197, 199, 36, 180, 100, 215, 107, 62, 89, 81, 111, 119, 32, 156, 214, 88, 183, 238, 18, 125, 231, 92, 127, 219, 138, 193, 141, 103, 37, 236, 157, 41, 158, 135, 120, 9, 250, 172, 106, 136, 2, 123, 247, 248, 26, 52, 54, 57, 204, 232, 7, 15, 140, 66, 245, 170, 144, 22, 203, 1, 56, 167, 34, 244, 137, 19, 225, 143, 6, 184, 10, 60, 151, 165, 91, 40, 133, 70, 128, 121, 220, 16, 152, 13, 58, 185, 254, 154, 198, 113, 160, 132, 206, 50, 122, 116, 192, 179, 153, 47, 95, 200, 112, 145, 5, 126, 105, 243, 164, 181, 146, 161, 129, 3, 48, 182, 189, 33, 148, 162, 69, 43, 234, 35, 39, 63, 150, 142, 61, 90, 64, 78, 42, 83, 21, 155, 168, 229, 96, 173, 208, 207, 221, 82, 242, 240, 27, 4, 186, 115, 17, 51, 159, 175, 75, 201, 44, 29, 218, 216, 108, 8, 99, 28, 102, 118, 24, 230, 195, 86, 226, 166, 11, 0, 171, 65, 228, 38, 223, 31, 67, 77, 49, 194, 124, 249, 222, 177, 252, 98, 235, 12, 210, 134, 233, 87, 255, 147, 149, 213, 25, 224]
INV_SBOX = [0]*256

for i in range(256):
    INV_SBOX[SBOX[i]] = i

for l in range(6,13):
    # è necessario installare xortool per eseguire questo comando
    subprocess.run(f"xortool -x {sys.argv[1]} -l {l} -b", shell=True, stdout = subprocess.DEVNULL)

    # xortool salva in questa cartella i suoi output,
    # tra cui il tentativo di decryption del ciphertext
    for file in os.listdir("xortool_out"):
        filename = os.fsdecode(file)
        if filename.endswith(".out"): 
            enc = open(f"xortool_out/{filename}", "rb").read()
            dec = bytes([INV_SBOX[x] for x in enc])
            try:
                dec = dec.decode()
                if all([x in string.printable for x in dec]):
                    print("flag{"+dec.split("|")[-1]+"}")
            except:
                pass
subprocess.run(f"rm -r xortool_out", shell=True, stdout = subprocess.DEVNULL)
```
