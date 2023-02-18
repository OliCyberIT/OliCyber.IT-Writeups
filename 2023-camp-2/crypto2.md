# OliCyber.IT 2023 - Simulazione training camp 2

## [crypto] Sempre più primi, sempre più strani (40 risoluzioni)

La challenge è una semplice cifratura con RSA in cui però il primo $q$ viene generato in maniera deterministica a partire da $p$. L'obiettivo è quello di fattorizzare $n$ sfruttando la generazione insicura di $q$. Per farlo ci sono due metodi principali.

### Soluzione 1

Indichiamo con $f$ la funzione che genera il primo $q$ a partire da $p$. Quindi abbiamo che

$$
q = f(p)\quad n = p\cdot f(p).
$$

Dato che $n$ dipende solo da $p$, possiamo cercare di studiare come varia in funzione di quest'ultimo. In particolare, se prendiamo $p_1$ e $p_2$ con $p_1 > p_2$, vogliamo capire in che relazione sono $q_1 = f(p_1)$ e $q_2 = f(p_2)$. Notiamo che, data la scrittura binaria di $p$, la scrittura in base $4$ di $q = f(p)$ è data dalle cifre binarie di $p$ incrementate di $2$. Ad esempio, considerando $p_1 = 911$ e $p_2 = 607$, i rispettivi $q$ che otteniamo, in base 4, sono

$$
p_1 = 1110001111_2\quad \rightarrow \quad q_1 = 3332223333_4,\\
p_2 = 1001011111_2\quad \rightarrow \quad q_2 = 3223233333_4.
$$

Quindi è possibile dedurne che, se $p_1 > p_2$ allora si ha $q_1 > q_2$ e quindi anche $n_1 > n_2$, dove $n_1 = p_1\cdot q_1 = p_1\cdot f(p_1)$ e $n_2 = p_2\cdot q_2 = p_2\cdot f(p_2)$. Quindi $n$ è una funzione crescente di $p$ (cioè aumentando il valore di $p$ aumenta anche il valore di $n$).\\
Questa osservazione permette di trovare il valore di $p$ attraverso una semplice ricerca binaria:

1. Prendiamo $\lfloor\frac{n}{2}\rfloor$ come valore iniziale di $p_0$
2. Calcoliamo $n_0 = p_0\cdot f(p_0)$
3. Se $n_0 < n$ vuol dire che il valore del primo $p$ che stiamo cercando è maggiore di $p_0$ altrimenti che è minore
4. A questo punto procediamo come nella ricerca binaria

```python
def binary_search(n, n_bits):
    a, b = 0, n
    p = ((a + b)//2)
    while n % p != 0 and p not in [1, n]:
        q = generate_second_prime(n_bits, p)
        if p*q < n:
            a, b = p, b
        else:
            a, b = a, p
        p = ((a + b)//2)
    return p, n//p
```

### Soluzione 2

Un'altra tecnica importante è quella di provare a costruire il primo $p$ bit per bit a partire dal meno significativo. Infatti, se consideriamo l'equazione $n = p\cdot q$ modulo $2^i$ abbiamo che

$$
n \mod{2^i} = (p \mod{2^i})\cdot (q \mod{2^i}),
$$

che è un'equazione sui primi $i$ bits di $p$ e di $q$. Tuttavia i primi $i$ bit di $q$ sono completamente determinati dai primi $i$ bit di $p$: in particolare $i$ cifre meno significative in base $4$ di $q$ corrispondono agli $i$ bit meno significativi di $p$ incrementati di $2 e ogni cifra in base $4$ corrisponde a $2$ bit. Ad esempio, se $p \mod{2^8} = 143 = 10001111_2$ abbiamo che

$$
q = 32223333_4 = 1110101011111111_2 = 11111111_2 \mod{2^8} = 255.
$$

Quindi possiamo procedere nel seguente modo per costruire $p$ bit per bit:

1. Sappiamo che i bit meno significativi di $p$ e $q$ sono entrambi $1$, essendo dispari, quindi partiamo con una lista di candidati data da un solo elemento:

```python
candidates = [1]
```

2. Per ogni candidato nella lista, verifichiamo quale possa essere il suo bit successivo. Dato un candidato `c` proviamo a testare come bit successivo sia $1$ che $0$. Per ciascuno di questi calcoliamo il $q$ corrispondente e verifichiamo se l'equazione $n \mod{2^i} = (p \mod{2^i})\cdot (q \mod{2^i})$ è valida. Se lo è aggiungiamo il nuovo candidato alla lista, altrimenti lo scartiamo.

```python
def search_with_candidates(n, n_bits):
    candidates = [1]
    for i in range(1, n_bits):
        new_candidates = []
        for c in candidates:
            for b in [0,1]:
                possible_p = c + b*2**i
                possible_q = generate_second_prime(i+1, possible_p)
                if ((possible_p*possible_q) % 2**(i+1)) == (n % 2**(i+1)):
                    new_candidates.append(possible_p)
        candidates = new_candidates[:]
    for c in candidates:
        if n%c == 0:
            return c, n//c
    raise Exception("Prime not found")
```

### Exploit

```python
from Crypto.Util.number import isPrime, long_to_bytes

e = 65537
n = 3637400592141233053318303969046830876095580140855488833015304207130621295563617144193449778130146429235997567238720206838185374871913987902223981111694348323418739467704699968504263728458229055653571394452433642631694996588693092104798191817534563009299334375307873871479302638441206077852699432268647990793663566966521773196973270814465617055277014656580584957078884192564762129155432028080595364741266629162254969520149534387434381244067539296823332241573408517050613560436344530372143239028799557682567433850259477291205708952194250569729667458741243914742703705791789934874751084544389695970001574563357092617545184676320826820159122325980498989599327069001659755319055027479902935187549483616076942681091178586529367283446330939129905925913286207341103644775278407859109072901754487888245126284059167729062259953149500477613275922923526130302966024792147916266067202117122767618564413984607610874376274884601265902009593
c = 1655355477614725612326422801205654877153080140934683428882259793084639106115888344087475378116701397163573359080667070466301581841165682635444225467621376879028265845200417675411957733990127166864545252527726341439173387141659284443969047753331664302512055816466043523216757941892216028198045457204121696059880460850993225208201818563800454385936127215140806793291869684379413641376098990886209220526244581344109561158979268164451880821578009427297389894607793649073020633897931021875052533975748762718599063074637040985914014817824594781556829792222319250544787513048629033230346849193900562970003140272836934559676170690791681457233204121884138289007560796468623852258130364150530365533423573124461481907646394604342186444873177972242783039921686397715848501012679703895403159284449510822928617975181794895132833879302286751191844491913951297282845352765885767209341868827516990441630639813899722532299502152935651006231333
n_bits = 1024

def generate_second_prime(n_bits, p):
    p_bits = (bin(p)[2:]).zfill(n_bits)
    increased_bits = [int(b) + 2 for b in p_bits]
    q = sum([int(d) * int(4**(n_bits - 1 - i)) for i,d in enumerate(increased_bits)])
    return q

def binary_search(n, n_bits):
    a, b = 0, n
    p = ((a + b)//2)
    while n % p != 0 and p not in [1, n]:
        q = generate_second_prime(n_bits, p)
        if p*q < n:
            a, b = p, b
        else:
            a, b = a, p
        p = ((a + b)//2)
    return p, n//p

def search_with_candidates(n, n_bits):
    candidates = [1]
    for i in range(1, n_bits):
        new_candidates = []
        for c in candidates:
            for b in [0,1]:
                possible_p = c + b*2**i
                possible_q = generate_second_prime(i+1, possible_p)
                if ((possible_p*possible_q) % 2**(i+1)) == (n % 2**(i+1)):
                    new_candidates.append(possible_p)
        candidates = new_candidates[:]
    for c in candidates:
        if n%c == 0:
            return c, n//c
    raise Exception("Prime not found")

# p, q = search_with_candidates(n, n_bits)
p, q = binary_search(n, n_bits)

assert p*q == n

phi = (p-1)*(q-1)
d = pow(e, -1, phi)
flag = long_to_bytes(pow(c, d, n)).decode()
print(flag)
```
