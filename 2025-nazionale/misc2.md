# OliCyber.IT 2025 - Competizione nazionale

## [misc] censored (59 solves)

C'è questo sito web che crea delle cartoline carine con il tuo nome e la flag sopra, dagli un'occhiata!

> Nota: il formato della flag è flag{[a-z_]+}

Site: [https://censored.challs.olicyber.it](https://censored.challs.olicyber.it)

Author: Lorenzo Catoni <@lorenzcat>

## Panoramica

Il sito genera delle immagini contenenti un nome specificato dall'utente concatenato alla flag. Tuttavia, i caratteri del testo sono "censurati": nell'immagine si vedono solamente dei rettangoli neri.

È molto evidente come i rettangoli abbiano larghezze diverse; ciò è dovuto al fatto che i caratteri stessi hanno larghezze differenti (ad esempio, la `i` è più stretta della `m`).

## Soluzione

Per risolvere l'esercizio è sufficiente specificare come nome utente l'intero alfabeto utilizzato nella flag (caratteri minuscoli, graffe e underscore). In questo modo è possibile ottenere una mappatura tra "larghezza → carattere". Successivamente, osservando le larghezze dei rettangoli corrispondenti alla flag, si può ricostruirla.
L'idea è semplice; l'unica complessità sta nello scrivere uno script in grado di calcolare correttamente le larghezze dei rettangoli neri.

## Exploit

```py
import io
import string
import sys

import requests
from PIL import Image

URL = f'{sys.argv[1]}/image?name={{}}'
len_flag = 32
cols = 17
dic = string.ascii_lowercase + '{}_'


response = requests.get(URL.format(dic))
image = Image.open(io.BytesIO(response.content))
pix = image.load()

black = (0, 0, 0)
nchar = len(dic) + len_flag
X = 50
y = 0

while pix[X, y] != black:
    y += 1
x = 0

widths = []
for char_idx in range(nchar):
    while pix[x, y] != black:
        x += 1
    xl = x

    while pix[x, y] == black:
        x += 1

    widths.append(x - xl)

    # go to new line
    if (char_idx  + 1) % cols == 0:
        while pix[X, y] == black:
            y += 1

        while pix[X, y] != black:
            y += 1
        x = 0

widths_dic = widths[:len(dic)]
widths_flag = widths[len(dic):]

assert len(widths_dic) == len(dic)
widths_map = {}
for w, c in zip(widths_dic, dic):
    assert w not in widths_map, "not unique"
    widths_map[w] = c

flag = ''.join(widths_map[wf] for wf in widths_flag)
print(flag)
```
