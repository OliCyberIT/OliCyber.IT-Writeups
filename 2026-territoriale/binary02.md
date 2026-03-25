# OliCyber.IT 2026 - Selezione Territoriale

## [binary] La mappa (76 solves)

Trova la strada giusta e avrai la flag!

Author: Alberto Carboneri <@Alberto247>

## Panoramica

L'obiettivo di questa challenge è estrarre una sequenza di caratteri da un programma a partire dalle informazioni presenti nella sua memoria al momento dell'esecuzione, senza necessariamente capirne in maniera approfondita l'assembly originario. La parola chiave è "mappa", che fa riferimento proprio a un "grafo" o "labirinto" salvato e hard-codato direttamente all'interno delle costanti del binario.

## Soluzione

Il binario implementa e modella in memoria un grafo orientato. Grazie a utility come `readelf` o aprendolo in librerie dedicate come `pwn.ELF` (pwntools) o in Ghidra, possiamo notare che il grafo è suddiviso logico-matematicamente in tre strutture di dati/array:

- `edge_from`: I nodi da cui partono gli archi.
- `edge_label`: I nomi / i caratteri contrassegnati ad ogni rispettiva transizione.
- `edge_to`: I nodi d'arrivo degli archi.

Essendo un grafo relativamente contenuto e, soprattutto, lineare (ogni nodo ha esattamente un solo successore), possiamo automatizzarne l'estrazione in maniera molto semplice. Con `pwntools` recuperiamo per intero questi tre array; da qui costruiamo in locale una semplice associazione nodo di partenza - (carattere, nodo di arrivo).
Identificando il nodo iniziale, il numero `0`, possiamo semplicemente iterare seguendo l'unico arco disponibile per ciascun nodo fino ad arrivare a quello finale, accumulando al contempo le label ad essi associate per formare la flag.

## Exploit

```python
from pwn import ELF

elf = ELF("./mappa")
addr_from = elf.sym["edge_from"]
addr_label = elf.sym["edge_label"]
addr_to = elf.sym["edge_to"]

n = 36
edge_from = list(elf.read(addr_from, n))
edge_label = list(elf.read(addr_label, n))
edge_to = list(elf.read(addr_to, n))

graph = {}
for i in range(n):
    graph[edge_from[i]] = (chr(edge_label[i]), edge_to[i])

node = 0
path_str = ""
while node != 36:
    label, to_node = graph[node]
    path_str += label
    node = to_node

print("Flag:", path_str)
```
