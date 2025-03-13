# OliCyber.IT 2025 - Selezione territoriale

## [misc] Trasportando file (77 risoluzioni)

Mi sono reso conto che i file che scambiavo coi miei amici non erano protetti. Ora invece mi sento più al sicuro!

Autore: Matteo Protopapa <@matpro>

## Panoramica

La challenge consiste in un pcap contenente prevalentemente traffico FTP. Da una prima analisi con Wireshark sembra che gli unici comandi eseguiti dopo il login siano di tipo `STOR`, quindi si tratta di upload di file. Il contenuto effettivo si può distinguere dai comandi FTP per il protocollo, che è FTP-DATA.

## Soluzione

Possiamo estrarre tutti i file scambiati durante la cattura del traffico con Wireshark, tramite la voce `File > Export Objects > FTP-DATA...` e quindi `Save All`. In alternativa, si può utilizzare il comando `tshark -r trasportando_file.pcap --export-objects "ftp-data,extracted"` sul terminale, per estrarre tutti gli oggetti nella cartella `extracted`.

Possiamo notare che tutti i file hanno un nome del tipo `immagine0.png_00.enc`, fatta eccezione per il file `encrypt_files.py`. I primi sonoo file cifrati, mentre l'ultimo è in chiaro:

```python
import sys
from hashlib import sha256
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad

if len(sys.argv) != 2:
    print(f'Usage: {sys.argv[0]} <file to encrypt>')
    exit(0)

CHUNK_SIZE = 13337

file_to_encrypt = sys.argv[1]
readfile = open(file_to_encrypt, 'rb')
content = readfile.read()
readfile.close()

chunks = [content[CHUNK_SIZE*i:CHUNK_SIZE*(i+1)] for i in range(len(content) // CHUNK_SIZE + 1)]

for i, chunk in enumerate(chunks):
    iv = b'\x00' * 16
    cipher = AES.new(sha256(file_to_encrypt.encode()).digest(), mode=AES.MODE_CBC, iv=iv)
    enc = cipher.encrypt(pad(chunk, AES.block_size))
    writefile = open(f'encrypted_chunks/{file_to_encrypt}_{i:02}.enc', 'wb')
    writefile.write(enc)
    writefile.close()
```

da cui possiamo ricavare l'informazione che in realtà vengono scambiate solo 5 immagini, ma in pezzi. Poiché abbiamo tutti i file cifrati, gli IV (fissati a 16 byte nulli) e le chiavi (il nome del file da recuperare), possiamo decifrare i vari chunk, concatenarli, e riottenere le immagini originali.

## Exploit

Il seguente exploit assume che gli oggetti FTP-DATA siano stati estratti nella cartella `extracted`.

```python
import os
from hashlib import sha256
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad


chunks = sorted(os.listdir('extracted'))


for i in range(5):
    img_chunks = list(filter(lambda x: f"immagine{i}" in x, chunks))

    imgfile = open(f'decrypted/immagine{i}.png', 'wb')

    for c in img_chunks:
        iv = b'\x00' * 16
        file_to_encrypt = f'immagine{i}.png'
        cipher = AES.new(sha256(file_to_encrypt.encode()).digest(), mode=AES.MODE_CBC, iv=iv)
        content = open(f'extracted/{c}', 'rb').read()
        dec = unpad(cipher.decrypt(content), AES.block_size)
        imgfile.write(dec)
    
    imgfile.close()
```
