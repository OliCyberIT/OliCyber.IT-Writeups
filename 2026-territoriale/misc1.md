# OliCyber.IT 2026 - Selezione Territoriale

## [misc] un vecchio dvd (34 solves)

ho trovato questo dvd in soffitta... ci sarà qualcosa di interessante dentro?

Author: Andrea Raineri <@Rising>

## Panoramica

L'attachment della challenge è un file iso di un DVD. Il file può essere aperto con un qualsiasi lettore dvd, come VLC, e viene mostrato un menu iniziale con 3 titoli. Nessuno dei 3 video raggiungibili dal menu contiene la flag.

La flag è contenuta in un quarto video, non raggiungibile dal menu, ma presente all'interno del DVD. Il video è nascosto in una directory che non viene mostrata dal menu, ma è comunque accessibile e riproducibile.

## Soluzione

1. Con VLC è possibile riprodurre direttamente la traccia video n.4 selezionandola dal menu "Riproduzione > Titolo".

2. Con 'lsdvd' si ottiene un elenco delle tracce video presenti nel DVD

```bash
$ lsdvd dvd.iso
Title: 01, Length: 00:00:07.834 Chapters: 01, Cells: 01, Audio streams: 01, Subpictures: 00
Title: 02, Length: 00:00:07.767 Chapters: 01, Cells: 01, Audio streams: 01, Subpictures: 00
Title: 03, Length: 00:00:12.734 Chapters: 01, Cells: 01, Audio streams: 01, Subpictures: 00
Title: 04, Length: 00:00:29.967 Chapters: 01, Cells: 01, Audio streams: 01, Subpictures: 01
```

Quindi con 'mplayer' è possibile riprodurre direttamente la traccia video n.4

```bash
$ mplayer dvd://4 -dvd-device dvd.iso
```
