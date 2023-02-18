# OliCyber.IT 2023 - Simulazione training camp 1

## [misc] IHC - I Hate Captcha! (37 risoluzioni)

Challenge remota che richiede la risoluzione di captcha entro un tempo limite. Per ogni risoluzione corretta il servizio invia un carattere della flag.

### Soluzione

Il servizio invia a caso 4 tipi diversi di captcha da risolvere:

1. Una semplice operazione aritmetica tra interi (+, -, \*, / (troncata))
2. Scrivere quali sono le lettere che occupano le posizioni richieste nella stringa fornita
3. Riscrivere la stringa fornita al contrario
4. Contare quante volte la lettera specificata compare nella stringa fornita

Non è possibile completare la challenge rispondendo a mano a causa del tempo limite. "uindi, si deve creare uno script che risolva tutti e quattro i tipi di captcha e continuare a risolverli fino a quando non si saranno ricevuti tutti i caratteri della flag

### Exploit

```python
#!/bin/python3

from pwn import *
import os

r = remote(os.getenv('HOST'), int(os.getenv('PORT')))
r.recvuntil(b'invio!')
r.sendline()
flag = ''
while True:
    data = r.recvuntil(b'Risposta: ')

    # Semplice operazione
    if b'risultato' in data:
        data = data.split(b'\n')[0].split(b' ')
        b = data[-1][:-1].decode()
        op = data[-2].decode()
        a = data[-3].decode()
        if op == '+':
            r.sendline(str(int(a) + int(b)).encode())
        elif op == '-':
            r.sendline(str(int(a) - int(b)).encode())
        elif op == '*':
            r.sendline(str(int(a) * int(b)).encode())
        else:
            r.sendline(str(int(a) // int(b)).encode())

    # Lettere in stringa
    elif b'Quali' in data:
        data = data.split(b'\n')[0].split(b'posizioni ')[1].split(b']')
        positions = eval(data[0].decode() + ']')
        word = data[1].split(b' ')[-1].decode()[:-1]
        res = ''
        for p in positions:
            res += word[p-1]
        r.sendline(res.encode())

    # Stringa al contrario
    elif b'Restituiscimi' in data:
        data = data.split(b'\n')
        parola = data[0].split(b' ')[-1]
        r.sendline(parola[::-1])

    # Conta occorrenze
    else:
        data = data.split(b'\n')
        lettera = data[0].split(b'lettera ')[1][0]
        parola = data[0].split(b' ')[-1][:-1]
        r.sendline(str(parola.count(lettera)).encode())
    data = r.recvuntil(b'\n').decode()
    if 'Corretto' in data:
        flag += data.split(' ')[-1]
    if '}' in flag:
        break

print(flag.replace('\n', ''))

```
