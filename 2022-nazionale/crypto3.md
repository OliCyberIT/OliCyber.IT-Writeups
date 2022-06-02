# OliCyber.IT 2022 - Competizione nazionale

## [crypto-3] OTP-as-a-Service 2 (10 risoluzioni)

`Sembra che la prima versione avesse una debolezza, ma di sicuro questa nuova release sarà sicurissima!`

La challenge cifra la flag utilizzando una specie di one time pad con numeri casuali.

### Soluzione

Le uniche differenze con la versione 1 della challenge sono che è stato tolto il seeding con il timestamp, ma è anche stata tolta la riduzione in modulo. In particolare, la funzione di encryption è $c_i = m_i+r_i$, dove $r_i$ è un numero casuale tra 0 e 255.

Avendo a disposizione il server remoto che cifra la flag quante volte vogliamo, possiamo condurre un'analisi statistica sui ciphertext restituiti.

L'osservazione fondamentale è che, poiché $0\le r_i\le 255$ e non c'è nessuna riduzione in modulo, allora vale $m_i\le c_i\le m_i+255$: per ogni indice, l'i-esimo numero restituito dal server appartiene sempre all'intervallo $[m_i, m_i+255]$.

In particolare, mandando abbastanza richieste possiamo essere quasi certi di ottenere almeno una volta $r_i=0$, ovvero $c_i=m_i$. Pertanto, interrogando molte volte il server e prendendo il minimo dei valori ottenuti per ogni carattere è molto probabile riuscire a ricostruire la flag.

Per avere una stima quantitativa, supponiamo di interrogare il server $M$ volte; per un singolo carattere, la probabilità di ottenere almeno una volta $r_i=0$ è pari a $$1-\left( \frac{255}{256} \right)^M$$

Dato che questo deve succedere per tutti e $L=71$ caratteri della flag, la probabilità di successo è del $$\left( 1-\left( \frac{255}{256} \right)^M \right)^L$$
Con la scelta di $M=2000$, questa probabilità viene del 97%.

### Exploit

```python
from pwn import remote


class Client:
    def __init__(self):
        self.r = remote("otp2.challs.olicyber.it", 12306)
        self.r.recvlines(4)

    def get_encrypted(self):
        self.r.sendline("e")
        res = self.r.recvline(False).decode()
        return list(map(int, res.split("-")))


c = Client()
n = 71
collected = [[] for _ in range(n)]
M = 2000
for _ in range(M):
    enc = c.get_encrypted()
    for i in range(n):
        collected[i].append(enc[i])

flag = ""
for i in range(n):
    guess = min(*collected[i])
    flag += chr(guess)

print(flag)
```
