# OliCyber.IT 2023 - Simulazione training camp 3

## [misc] GES - Gabibbo Encoding Standard (36 risoluzioni)

La challenge consiste in uno script python che implementa un encoding custom per stringhe. Viene fornita la flag encodata con lo script.

### Soluzione

L'encoding della stringa esegue diversi passaggi sull'input, in ordine:

1. I due cicli tra le righe 39 e 42 riordinano i caratteri della stringa in modo che siano presenti prima tutti quelli in posizioni dispari, poi quelli in posizioni pari
2. Alla riga 43 inverte la stringa ordinata e la trasforma in bytes per il passaggio successivo
3. La funzione `roba` calcola il base 18 dell'argomento passatogli
4. Stampa il base 18 utilizzando come cifre i versi della sigla di `Paperissima Sprint` separati da un punto e virgola

Quindi, per ottenere al flag decodificata basta reversare a ritroso, dal punto 4 all'1, i passaggi eseguiti dall'encoding

### Exploit

```python
import math

m = """Splash!
E sotto l'onda, profonda
Insieme io e te
Che bello i pesci
Stare a guardare
Che baraonda, gioconda
Pepperrepeppè
È il pesce tromba che sta a suonare
Seppie e acciughe con te (ballano)
Scampi e orate con me (saltano, danzano)
(E poi) la sera
(La lu) na piena
Ecco che inizia il gran galà
(Le acciughe) in festa
(Perdon) la testa
Con seppie e scampi saltan già
Nella padella con la pastella
Che fritto misto per noi!""".split('\n')

enc = "E sotto l'onda, profonda;Scampi e orate con me (saltano, danzano);Scampi e orate con me (saltano, danzano);(Le acciughe) in festa;Ecco che inizia il gran galà;Ecco che inizia il gran galà;(Le acciughe) in festa;E sotto l'onda, profonda;È il pesce tromba che sta a suonare;Scampi e orate con me (saltano, danzano);Ecco che inizia il gran galà;Insieme io e te;Che bello i pesci;(E poi) la sera;Scampi e orate con me (saltano, danzano);Ecco che inizia il gran galà;E sotto l'onda, profonda;Insieme io e te;(Perdon) la testa;(Le acciughe) in festa;Insieme io e te;È il pesce tromba che sta a suonare;(Le acciughe) in festa;Che baraonda, gioconda;Insieme io e te;Che bello i pesci;(Le acciughe) in festa;È il pesce tromba che sta a suonare;Stare a guardare;(La lu) na piena;Che baraonda, gioconda;Ecco che inizia il gran galà;Che bello i pesci;Splash!;Che baraonda, gioconda;Pepperrepeppè;È il pesce tromba che sta a suonare;Insieme io e te;Pepperrepeppè;Stare a guardare;Ecco che inizia il gran galà;Che fritto misto per noi!;(Le acciughe) in festa;Stare a guardare;È il pesce tromba che sta a suonare;Che fritto misto per noi!;Seppie e acciughe con te (ballano);Pepperrepeppè;Che baraonda, gioconda;È il pesce tromba che sta a suonare;Stare a guardare;È il pesce tromba che sta a suonare;Con seppie e scampi saltan già;E sotto l'onda, profonda;Ecco che inizia il gran galà;Scampi e orate con me (saltano, danzano);Stare a guardare;Nella padella con la pastella;Pepperrepeppè;Ecco che inizia il gran galà;Con seppie e scampi saltan già;Che fritto misto per noi!;(Le acciughe) in festa;Insieme io e te;Che fritto misto per noi!;Con seppie e scampi saltan già;Stare a guardare;Stare a guardare;Nella padella con la pastella;(La lu) na piena;Con seppie e scampi saltan già;E sotto l'onda, profonda;Che baraonda, gioconda;(Le acciughe) in festa;(E poi) la sera;E sotto l'onda, profonda;Insieme io e te;Ecco che inizia il gran galà;Scampi e orate con me (saltano, danzano);Con seppie e scampi saltan già;Scampi e orate con me (saltano, danzano);(Perdon) la testa;E sotto l'onda, profonda;Ecco che inizia il gran galà;Pepperrepeppè;Con seppie e scampi saltan già;Che bello i pesci;È il pesce tromba che sta a suonare;(Le acciughe) in festa;È il pesce tromba che sta a suonare;(La lu) na piena;Stare a guardare;Nella padella con la pastella;(La lu) na piena;Con seppie e scampi saltan già;(Le acciughe) in festa;Insieme io e te;Nella padella con la pastella;Pepperrepeppè;Con seppie e scampi saltan già;(Perdon) la testa;Con seppie e scampi saltan già;Insieme io e te;Che bello i pesci;Seppie e acciughe con te (ballano);Scampi e orate con me (saltano, danzano);Ecco che inizia il gran galà;Scampi e orate con me (saltano, danzano);Che bello i pesci;Splash!;(E poi) la sera;Ecco che inizia il gran galà;Scampi e orate con me (saltano, danzano);Con seppie e scampi saltan già;Pepperrepeppè;Splash!;Pepperrepeppè;Scampi e orate con me (saltano, danzano);Con seppie e scampi saltan già;(Perdon) la testa;(La lu) na piena;(Perdon) la testa;(E poi) la sera;Con seppie e scampi saltan già;Nella padella con la pastella".split(';')
indexes = []
n = 0

# Trasformo le "cifre" del base 18 in valori decimali
for line in enc:
    indexes.append(m.index(line))

# Converto il base 18 in base 10
for i, a in enumerate(indexes[::-1]):
    n += a*18**i

# COnverto in binario per ottenere la stringa reversata e la reverso per ottenere quella ordinata
s = n.to_bytes((math.log(n, 2)/8).__ceil__(), 'big').decode()[::-1]
res = ''

# Riordino per ottenere l'input iniziale
a = s[:(len(s)/2).__ceil__()]
b = s[(len(s)/2).__ceil__():]

for l in zip(a,b):
    print(l[0]+l[1], end='')
if len(s) % 2 != 0:
    print(a[-1])
```
