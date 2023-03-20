# OliCyber.IT 2023 - Selezione territoriale

## [crypto-2] IndovinANDo segreti (70 risoluzioni)

La challenge chiede di ricostruire una stringa segreta `s` partendo dai risultati di `s & P[i]`, dove `P` è una permutazione dei 256 possibili valori di un byte.

### Soluzione

La prima osservazione è che per ogni carattere `c` della stringa vale che `c & 0xff == c`; se riuscissimo quindi a fare la query tale per cui `P[i] = 0xff` avremmo recuperato la stringa segreta.

Tuttavia, possiamo chiedere tutti i valori tra 0 e 255, e siamo sicuri di farci rispondere esattamente una volta con il byte `0xff`; in particolare, possiamo recuperare il carattere originale prendendo il byte che ha più uni nella sua scrittura binaria tra quelli che ci vengono risposti.

Questo perché, detto `hw` il peso di Hamming di un byte, per ogni byte `b` vale `hw(c & b) <= hw(c) = hw(c & 0xff)`: infatti nelle posizioni in cui `b` ha un 1, `c & b` ha lo stesso bit di `c`, mentre dove `b` ha uno 0, sicuramente `c & b` avrà uno 0.

### Exploit

```python
from pwn import remote


def hw(x):
    return bin(x).count("1")


r = remote(HOST, PORT)
r.recvlines(2)

for _ in range(10):
    r.recvline()
    sol = [0]*8
    for i in range(256):
        r.sendlineafter(">", str(i))
        data = r.recvline(False).decode().strip()
        res = bytes.fromhex(data)
        for j in range(8):
            if hw(res[j]) > hw(sol[j]):
                sol[j] = res[j]
    ans = bytes(sol).hex()
    r.sendlineafter(">", "g")
    r.sendlineafter("?", ans)

flag = r.recvline(False).decode()
print(flag)

```
