# OliCyber.IT 2024 - Finale Nazionale

## [misc] Around the World (11 risoluzioni)

Il Gabibbo è appena tornato da un lungo viaggio: ha seguito un percorso interessante, queste sono le foto che ha scattato.

Il formato della flag è: `FLAG{[A-Z]+}`.

Author: Lorenzo Catoni <@lorenzcat>

## Panoramica

La challenge fornisce un file zip contenente una serie di immagini.

## Soluzione

Le immagini contengono metadati di localizzazione e tempo, che possono essere letti utilizzando, per esempio, `exiftool`.

```bash
$ exiftool -DateTimeOriginal -GPSLatitude -GPSLongitude around-the-world/02dc17ab.jpg
Date/Time Original              : 2023:04:19 09:30:00
GPS Latitude                    : 45 deg 0' 0.00"
GPS Longitude                   : 14 deg 46' 55.20"
```

Dobbiamo tracciare queste posizioni e collegarle in ordine cronologico. L'unico avvertimento è che, per rendere il grafico più leggibile, è più conveniente collegare solo i punti relativi allo stesso giorno.

## Exploit

```py
import itertools
import os
import re
import subprocess
from dataclasses import dataclass
from datetime import datetime
from pathlib import Path
from typing import List
import sys

import matplotlib.pyplot as plt

if len(sys.argv) != 2:
	print("Usage: python solution.py <input_dir>")
	sys.exit(1)

INPUT_DIR = Path(sys.argv[1])

_, _, photos = next(os.walk(INPUT_DIR))
print(len(photos))


@dataclass
class Metadata:
    time: datetime
    lat: float
    lon: float

metadatas: List[Metadata] = []


def degstr_to_dec(degstr: str) -> float:
    """Converts a degree string to decimal
    es. 37 deg 37' 6.00" -> 37.61833333333333
    """
    dms = re.split(r"deg|'|\"", degstr)[:3]
    d, m, s = list(map(float, dms))
    return d + m / 60 + s / 3600

for photo in photos:
    print(photo)

	# parse time, latitute and longitude from exif data
    exif_command = f"exiftool -DateTimeOriginal -GPSLatitude -GPSLongitude {INPUT_DIR / photo}"
    out = subprocess.check_output(exif_command.split()).decode().split("\n")

    time = datetime.strptime(out[0].split(": ")[1], "%Y:%m:%d %H:%M:%S")
    lat = degstr_to_dec(out[1].split(": ")[1])
    long = degstr_to_dec(out[2].split(": ")[1])

    metadatas.append(Metadata(time, lat, long))

metadatas.sort(key=lambda x: x.time)

# groupby same day and plot
for _, ms in itertools.groupby(metadatas, key=lambda x: x.time.day):
    ms = list(ms)
    plt.plot([m.lon for m in ms], [m.lat for m in ms], "-o")

plt.show()
```