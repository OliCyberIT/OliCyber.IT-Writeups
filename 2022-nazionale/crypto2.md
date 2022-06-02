# OliCyber.IT 2022 - Competizione nazionale

## [crypto-2] Rompi Schemi Asimmetrici (38 risoluzioni)

`Mi hanno chiesto di implementare RSA per una nuova libreria di crittografia. È immediata da usare, spero sia anche sicura...`

La challenge cifra la flag con un'implementazione "textbook" di RSA, ma con un esponente pubblico molto partcolare.

### Soluzione

Osserviamo che lo script sta implementando RSA "textbook" con una generazione dei primi sicura; tuttavia la classe ha come default `e=-1`, e nell'istanziarla questo valore di default non viene sovrascritto.

Questo vuol dire che la funzione di encryption di RSA, che in generale è della forma $$c\equiv m^e\pmod n,$$ in questo caso diventa $$c\equiv m^{-1}\pmod n,$$ ovvero il testo cifrato è semplicemente l'inverso moltiplicativo del messaggio modulo $n$.

La sicurezza di RSA deriva dal fatto che è difficile estrarre radici $e$-esime per $e$ generici; ma in questo caso, la funzione inversa di $c = m^{-1}$ è $m=c^{-1}$.

Pertanto è sufficiente calcolare l'inverso moltiplicativo del ciphertext modulo $n$ (che è pubblico) per ottenere la flag.

### Exploit

```python
from Crypto.Util.number import inverse, long_to_bytes

N = 15761696848277146240220170349153208659450329266072524719236431720789636378853244291428358990415259673549517077469211924515483732020527197787851770155686589521637147514344912873609085293046613003246243830498095228858516120792708944055254822701488120684764826007679159424664423368766660756415494990401731476945297028728352184255206345834923734785230215986465807425597155477517063934877179576460995237796181533624257456816765006880573493162655849893840723838895424520475949085458743325302944280880961705884935902211039304686139566410299263827476969433755116540895058638630865634579944248572615610303648070774889858467017
flag_enc = 0x7681bb9a7eac3a339d193eda4b918935145266bc947cdf8296c374ece9f112dfdc2573af902c48b2ea1e87c80f5a64ae809cfb552269793eb1a022ba771f5845d0ecb8e517dff03db7ec17be348b5b8886bb77dc5d5fceaede635a913cc492e6573603b97d9b054e027f0fcb2549b31f03d4425534c64649d7517f0ffcbea3778cfbfb8e6ab4c4f3358ae3719e75646dcdfd1176421f37700205ad279a339cf1516cc4566ab5aa400511de05b5b0148b4efc469d0c916fcf64830cfd08cd64b0708dc11ca3449d52406dbaacba975d4043c06fc54c80462f0535f4b4334b12c46e665b809cfc6b20af4c7e827ce431e66725b12210b2ad521fa27de75d91d570


m = inverse(flag_enc, N)
flag = long_to_bytes(m)
print(flag)
```
