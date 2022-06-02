# OliCyber.IT 2022 - Competizione nazionale

## [crypto-4] RSwAp (3 risoluzioni)

`Perché generare due primi quando ce ne basta uno solo?`

La challenge consiste in un'implementazione "textbook" di RSA con una generazione particolare dei due numeri primi.

### Soluzione

La challenge genera un numero primo grande `p` e scambia due delle sue cifre decimali per generare un altro primo `q`. Dopo questo passaggio, esegue semplicemente la cifratura delle flag con RSA.

Per risolvere la challenge, notiamo che `q = p + 10^h*a + 10^k*b - 10^k*a - 10^h*b`, dove `a` e `b` sono le cifre scambiate (numeri da 0 a 9) e `h` e `k` sono le loro posizioni nella scrittura decimale di `p`. Sperimentalmente possiamo notare che un primo di 1024-bit ha 308 o 309 cifre, quindi `h` e `k` sono limitati superiormente da questo valore. Possiamo quindi semplicemente provare tutti i valori possibili di `a`, `b`, `h` e `k` (circa `10^6` tentativi totali) e per ognuno di essi risolvere l'equazione di secondo grado `p*q - n = 0` sostituendo in `q` l'espressione ricavata precedentemente e dipendente da `p`, `a`, `b`, `h` e `k`

### Exploit

```py
from gmpy2 import mpz, isqrt, next_prime
from Crypto.Util.number import long_to_bytes

def fac(n, len_p = 310):
    a = 1
    c = -n
    for x in range(10):
        for y in range(10):
            for h in range(len_p):
                for k in range(len_p):
                    b = 10**h*y+10**k*x-10**h*x-10**k*y
                    sq = isqrt(mpz(b**2-4*a*c))
                    p = int((-b + sq) // (2*a))
                    if n%p == 0 and p != n and p>1:
                        return p
    return -1


n = 9700624973335227055834821387578165820987729817710621527082447155621161414295958099546312097933523578863057993140379629780916269670655470321976734594875144696433530227361137525650790448084448289047204800420531636552990768244095520038285748691537342063595688661621589282117407882397058048167638703912241478051849281193195487402061635333315670227116368030820989272197103085229555972573665288381582068869270978768295751239967706497051433039255249462088379725705374118278747647279659599170149326364777103028984912684556440494924611905789226845891638154706316392010947789075975455446965196233425530921819310799161323107809
enc = 8255580707896644432561956914424511471156334555984014637394268024014293285137846892082431680072999329834245851671226254792610938021174324290279689177673407973448165108483400895114345223920594207515074785977239456887513632895162796944772179710008119306496435999952704488049608909998364713995991189159984715391344799912651466947076818542145290798936009114564571357839411521115728712609700550218526388693065678870553612075888445831063049794685793351197809113461654235636151015053726345333633951060839323791405505967387608467727815386445368427526327272024540927329442616358313519898374951339720776062847341416474879607959

p = fac(n)
e = 65537
assert p != -1
q = n//p

phi = (p-1)*(q-1)
d = pow(e, -1, phi)
print(long_to_bytes(pow(enc, d, n)))
```
