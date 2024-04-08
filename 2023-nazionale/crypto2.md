# OliCyber.IT 2023 - Competizione Nazionale

## [crypto] Poly(not)1305 (14 risoluzioni)

La challenge permette di registrare utenti, che vengono autenticati tramite un MAC custom. L'obiettivo è riuscire a forgiare un cookie da admin.

### Soluzione

Il MAC implementato è molto simile a Poly1305. In particolare per autenticare un messaggio $$m_0,m_1,\dots,m_n$$ viene calcolato il polinomio $$m_0 + m_1\cdot k +\dots + m_n\cdot k^n\pmod q$$, dove $$k$$ è la chiave segreta del MAC.

Poiché possiamo far firmare cookies arbitrari, abbiamo il pieno controllo dei coefficienti del polinomio; dato che sappiamo anche il modulo, si possono trovare le radici del polinomio, tra cui la chiave che ci permetterà di firmare cookies arbitrari.

### Exploit

```python
r = remote(HOST, PORT)
r.recvlines(1)
r.sendlineafter("> ", "2")
r.sendlineafter("? ", "a")
cookie = r.recvline(False).decode()
data, tag = cookie.split(".")
data = data.encode()
tag = int(tag)
blocks = [data[i*BLOCK_LEN:(i+1)*BLOCK_LEN] for i in range(len(data)//BLOCK_LEN + 1)]
bi = [int.from_bytes(b, byteorder="big") for b in blocks]
a, b, c = bi[0], bi[1], bi[2]-tag
delta = (b*b-4*a*c)%q
sqr = tonelli(delta, q)
k = (-b+sqr)*modinv(2*a, q) % q

msg = "admin=True"
data = msg.encode()
blocks = [data[i*BLOCK_LEN:(i+1)*BLOCK_LEN] for i in range(len(data)//BLOCK_LEN + 1)]
bi = [int.from_bytes(b, byteorder="big") for b in blocks]
tag = (bi[0]*k+bi[1]) % q

r.sendlineafter("> ", "1")
r.sendlineafter(": ", f"{msg}.{tag}")
resp = r.recvline(False).decode()
flag = r.recvline(False).decode()

print(flag)
```
