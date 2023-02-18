# OliCyber.IT 2023 - Simulazione training camp 2

## [misc] Gab-chan.png (5 risoluzioni)

La challenge consiste in un file `png` in cui è stata nascosta la flag tramite steganografia.

### Soluzione

Caricando la foto su un sito come [aperisolve](https://www.aperisolve.com/) si possono trovare 3 dati nascosti:

1. La password `VjyQ[6M8WFx[sLCT`
2. La presenza di un file chiamato `flag.txt` salvato nella foto tramite steganografia `lsb` (least significant bit)
3. Il commento `!flag: https://bit.ly/s3cr3t_m3s5ag3` il cui link redirecta a un Rickroll

Utilizzando un tool come [stego-lsb](https://github.com/ragibson/Steganography) si può estrarre il file nascosto. Il file è uno zip protetto dalla password `VjyQ[6M8WFx[sLCT` che contiene il file `flag.txt` con la flag

### Exploit

```sh
#! /bin/sh
stegolsb steglsb -r -i Gab-chan.png -o output_file.zip -n 2 && unzip -P 'VjyQ[6M8WFx[sLCT' output_file.zip && cat flag.txt
```
