# OliCyber.IT 2022 - Competizione territoriale

## [network-1] Daring New State (54 risoluzioni)

La challenge mette a disposizioneun server DNS a cui è possibile fare query. Tra le informazioni contenute nel server, è nascosta la flag, divisa in più parti. Per ricortruirla interamente è necessario seguire gli indizi forniti.

### Soluzione

Iniziamo cercando nei campi relativi al dominio fornito in descrizione (`the.flag`): è presente il record `here.is.the.flag` di tipo NS.

Andando ora a visualizzare i record relativi al dominio `here.is.the.flag`, ci saranno due record, uno di tipo A e uno di tipo TXT. Ci interessa il secondo, in quanto il contenuto è `flag{master_... look at the canonical name of base64decode.the.flag`.

Seguendo l'indizio fornito, scopriamo il record CNAME relativo a `base64decode.the.flag`: `Li4ub2ZfZG5zXzw+fSByZXBsYWNlIDw+IHdpdGggaXAgb2YgZDgxODc3NmQ3.the.flag`. Decodificando il base64, otteniamo `...of_dns_<>} replace <> with ip of d818776d7`.

L'ultimo step riguarda il record A di `d818776d7.the.flag`: l'indirizzo IP ad esso collegato è `127.254.13.25`, completando così la flag.

### Exploit

```python
#!/bin/env python3

import DNS  # py3DNS
import base64 as b64

server = 'newstate.challs-terr.olicyber.it'
port = 12008

flag = ''

the_flag = DNS.Request(server=[
                       server], port=port, name='the.flag', qtype='any', protocol='tcp').req().answers

assert any(_['data'] == 'here.is.the.flag' for _ in the_flag)

here_is_the_flag = DNS.Request(server=[
                               server], port=port, name='here.is.the.flag', qtype='any', protocol='tcp').req().answers
txt = list(filter(lambda x: x['typename'] == 'TXT', here_is_the_flag))[
    0]['data'][0].decode()
flag += txt.split('...')[0]

base64decode_the_flag = DNS.Request(
    server=[server], port=port, name='base64decode.the.flag', qtype='CNAME', protocol='tcp').req().answers
cname = base64decode_the_flag[0]['data']
flag += b64.b64decode(cname.split('.')[0]
                      ).decode().split('...')[1].split(' ')[0]


d818776d7_the_flag = DNS.Request(
    server=[server], port=port, name='d818776d7.the.flag', qtype='A', protocol='tcp').req().answers
a = d818776d7_the_flag[0]['data']
flag = flag.replace('<>', a)

print(flag)
```

Alternativamente, è possibile usare il comando `dig`, di cui riportiamo solo la prima query in quanto bastano poche modifiche per seguire tutti i passaggi:

```bash
dig @newstate.challs-terr.olicyber.it -p 12008 the.flag any
```
