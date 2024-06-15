# OliCyber.IT 2024 - Finale Nazionale

## [misc] Kinda diffusion (59 risoluzioni)

In IA, i modelli di diffusione aggiungono rumore all'immagine e poi lo eliminano.
Io ho aggiunto il rumore, tu rimuovilo!

Author: Aleandro Prudenzano <@drw0if>

## Panoramica

L'immagine viene elaborata aggiungendo un numero casuale nell'intervallo [0,255] a ogni pixel.

## Soluzione

Il seed del generatore di numeri casuali è anche generato casualmente nell'intervallo [0,255], quindi possiamo eseguire un attacco brute-force su tutti i seed eseguendo l'algoritmo al contrario. Per velocizzare il processo, possiamo utilizzare `multiprocessing` per "decrittare" più immagini contemporaneamente; alla fine, solo un'immagine avrà senso e conterrà la flag.

## Exploit
```python
from PIL import Image
import random
from multiprocessing import Pool as ThreadPool
import sys

def denoise(x):
    image, seed = x
    random.seed(seed)

    for x in range(image.width):
        for y in range(image.height):
            p = image.getpixel((x,y))
            p = tuple([(c - random.randint(0, 255))%256 for c in p])
            image.putpixel((x,y), p)

    print(f"Done with {seed}")
    image.save(f"output/flag-{seed}.png")

def main(filepath):
    img = Image.open(filepath)

    p = ThreadPool(16)
    p.map_async(denoise, [(img.copy(), s) for s in range(256)]).get(0xffff)


if __name__ == '__main__':
    if len(sys.argv) != 2:
        print(f"Usage: {sys.argv[0]} <image>")
        sys.exit(1)
    
    main(sys.argv[1])
```
