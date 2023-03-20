# OliCyber.IT 2023 - Selezione territoriale

## [network-1] Flag interceptor (176 risoluzioni)

La cattura fornita dalla challenge contiene le comunicazioni di 50 client verso lo stesso server. I vari client mandano delle stringhe, un carattere per volta, chiudendo e riaprendo la connessione ad ogni carattere inviato. Pertanto, ogni flusso TCP continene un solo carattere inviato. Come riporta la descrizione, solo una delle stringhe inviate rispetta il formato della flag.

### Soluzione

Ė possibile analizzare il contenuto della cattura con uno script (per comodità). L'obiettivo è quello di ricostruire le stringhe inviate separando i caratteri per indirizzo IP. Una volta ottenute tutte le stringhe filtriamo quelle che non rispettano il formato della flag, ottenendo un solo risultato che è effettivamente la flag.

### Exploit

```python
#!/bin/python3

import pyshark
from collections import defaultdict

streams = defaultdict(str)

cap = pyshark.FileCapture('attachments/flag-interceptor.pcap')

for p in cap:
    if 'data' in p:
        ip = p.ip.src
        data = p.data.data
        streams[ip] += bytes.fromhex(data).decode()[:-1]

for addr in streams:
    stream = streams[addr]
    if stream.startswith('flag{') and stream.endswith('}'):
        print(stream)
```
