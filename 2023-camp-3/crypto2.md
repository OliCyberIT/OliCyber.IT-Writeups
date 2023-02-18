# OliCyber.IT 2023 - Simulazione training camp 3

## [crypto] Xorn't (8 risoluzioni)

Il codice della challenge cifra la flag sommandoci delle OTP (modulo 2 elevato alla lunghezza della flag).

Per ottenere le OTP il codice moltiplica una chiave random lunga tanto quanto la flag per i numeri interi da 2 a 9 compresi.

### Soluzione

Le OTP non sono random (sono tutte in funzione di uno stesso dato, la chiave), inoltre l'operazione di cifratura non è uno _xor_.

Prendendo ad esempio le prime due cifrature (che corrispondono a $i = 2$ e $i = 3$), si ha che:

$$
\begin{cases}
\text{enc2} \equiv \text{FLAG} + 2 \cdot \text{key} \pmod{2^{\text{nbits}}} \\
\text{enc3} \equiv \text{FLAG} + 3 \cdot \text{key} \pmod{2^{\text{nbits}}}
\end{cases} \implies \begin{cases}
3 \cdot \text{enc2} \equiv 3 \cdot \text{FLAG} + 6 \cdot \text{key} \pmod{2^{\text{nbits}}} \\
2 \cdot \text{enc3} \equiv 2 \cdot \text{FLAG} + 6 \cdot \text{key} \pmod{2^{\text{nbits}}}
\end{cases}
$$

Dunque:

$$
\begin{align*}
3 \cdot \text{enc2} - 2 \cdot \text{enc3} &= 3 \cdot \text{FLAG} + 6 \cdot \text{key} - (2 \cdot \text{FLAG} + 6 \cdot \text{key}) \pmod{2^{\text{nbits}}} \\
&= \text{FLAG} \pmod{2^{\text{nbits}}} \\
&= \text{FLAG}
\end{align*}
$$

### Exploit

```python
#da encryptions.txt
enc2 = 2436308828056147886221436210996908065048757263693358568640090233041496519089249523516860504425
enc3 = 1985342668282272842808231966973340994107591895372667735999872511529407486711339603111799008223

flag = 3*enc2 - 2*enc3
print(flag.to_bytes(flag.bit_length()//8 + 1, 'big').decode())
```
