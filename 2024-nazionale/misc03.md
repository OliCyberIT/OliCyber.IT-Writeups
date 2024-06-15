# # OliCyber.IT 2024 - Finale Nazionale

## [misc] so far so good (9 risoluzioni)

Sono lontano da casa e ho inviato un segreto al mio Arduino tramite la rete... spero che nessuno lo scopra!

Author: Andrea Raineri <@Rising>

## Panoramica

La challenge consiste in uno sketch Arduino che legge un segreto dalla porta seriale e lo memorizza in una variabile globale. La connessione alla scheda Arduino viene eseguita tramite la porta seriale USB, quindi esposta sulla rete IP locale utilizzando il modulo kernel `usbip` di Linux.

Vengono forniti due allegati:
- Sketch Arduino eseguito sulla scheda
- Cattura del traffico TCP/IP tra il client e la scheda Arduino

## Soluzione

Analizzando lo sketch Arduino possiamo vedere che il segreto viene trasmesso criptato con ChaCha20 dopo uno scambio di chiavi. Possiamo estrarre la chiave e il nonce dalla cattura di rete e quindi decrittare il segreto.

Per estrarre i messaggi del protocollo personalizzato ricevuti dalla scheda Arduino, dobbiamo:

1. Estrarre i pacchetti USB BULK-IN/BULK-OUT dal flusso TCP nella cattura di rete
2. Estrarre i messaggi del protocollo personalizzato dai pacchetti USB

Dopo aver estratto tutti i messaggi,

3. Estrarre la `chiave` dal messaggio con opcode `0x74`
4. Estrarre il `nonce` dal messaggio con opcode `0x11`
5. Recuperare il `ciphertext` concatenando il contenuto di tutti i messaggi con opcode `0xC1`
6. Decrittare la flag con `ChaCha20(key, nonce, ciphertext)`

### Soluzione alternativa (proposta)
La soluzione proposta sfrutta la piccola dimensione del dump del traffico di rete, che permette di tentare il brute-forcing di tutte le posizioni in cui si trovano gli opcode dei protocolli personalizzati senza la necessità di analizzare ed estrarre i pacchetti USB all'interno dei segmenti TCP. Dopo aver cercato tutti i potenziali messaggi KEYSET e NONCESET, tutte le combinazioni vengono testate per inizializzare il cifrario e cercare i messaggi STORE contenenti la flag criptata.


## Exploit

```python
import pyshark
from collections import defaultdict
from Crypto.Cipher import ChaCha20
import sys

KEYSET    = 0x74
NONCESET  = 0x11
STORE     = 0xC1
RESET     = 0xB8
ACK       = 0x14
ERROR     = 0xFF

streams = defaultdict(lambda: b"")
cap = pyshark.FileCapture(f"dump.pcapng")

for p in cap:
    try:
        i = int(p.tcp.stream)
        streams[i] += bytes.fromhex(p.data.data)
    except:
        pass
cap.close()

stream = streams[0]

kidxs = [i for i, x in enumerate(stream) if x == KEYSET]
nidxs = [i for i, x in enumerate(stream) if x == NONCESET]
print(f'{kidxs = }')
print(f'{nidxs = }')


def read():
    global stream
    s = stream[0]
    stream = stream[1:]
    return s

class Message():
    def __init__(self) -> None:
        self.code = None
        self.param = None
        self.len = None
        self.data = None
    
    def read(self):
        self.code = read()
        self.param = read()
        self.len = read()
        self.data = []
        for i in range(self.len):
            self.data.append(read())

    def handle(self):
        global KEY, IV, cipher, keyset, nonceset, cipherset
        match self.code:
            case KEYSET:
                for i in range(32):
                    KEY[i] = self.data[i]
                keyset = True
                cipherset = False
            case NONCESET:
                for i in range(8):
                    IV[i] = self.data[i]
                nonceset = True
                cipherset = False
            case STORE:
                if not keyset or not nonceset:
                    return
                if not cipherset:
                    cipherset = True
                    cipher = ChaCha20.new(key=bytes(KEY), nonce=bytes(IV))
                dec = cipher.decrypt(bytes(self.data))
                for i in range(self.len):
                    FLAG[10 * self.param + i] = dec[i]
                if -1 not in FLAG:
                    print(bytes(FLAG))
                    return True
            case RESET:
                KEY = [0] * 32
                IV = [0] * 8
                keyset = False
                nonceset = False
                cipherset = False

_stream = stream[:]
for ki in kidxs:
    for ni in nidxs:
        if ki > ni:
            continue

        print('key index:', ki, '\tnonce index:', ni)

        KEY = [-1] * 32
        IV = [-1] * 8
        FLAG = [-1] * 40
        keyset = False
        nonceset = False
        cipherset = False
        cipher = None

        try:
            stream = stream[ki:]
            m = Message()
            m.read()
            m.handle()

            stream = stream[ni-ki-35:]
            m = Message()
            m.read()
            m.handle()

            while len(stream)>0:
                ni = stream.index(NONCESET) if NONCESET in stream else 99999
                si = stream.index(STORE) if STORE in stream else 99999
                next_idx = min([ni, si])
                stream = stream[next_idx:]
                m = Message()
                m.read()
                if m.handle():
                    break
        except:
            print('Exception')
            pass
        stream = _stream[:]
```