# OliCyber.IT 2026 - Selezione Territoriale

## [web] Rich Notes (14 solves)

A volte è utile poter prendere note con un minimo di formattazione...

Sito: [https://rich-notes.challs.olicyber.it/](https://rich-notes.challs.olicyber.it/)

Author: Lorenzo Leonardini <@pianka>

## Panoramica

La challenge si presenta come un semplice sito di note, su cui è possibile registrarsi, creare una nota, e condividere link a un admin.

Le note permettono di inserire un po' di formattazione, tra cui la possibilità di inserire testo in grassetto, corsivo, link, e immagini.

## Soluzione

Nell'inserire immagini, il backend verifica se nell'URL è stato inserito un carattere `"` per uscire dall'attributo `src` e inserire event handler per eseguire JavaScript.

Tuttavia, questa sanitizzazione non viene effettuata per i link. Si può cercare un evento utilizzabile con i link, e in particolare si nota come è possibile triggerare l'esecuzione dell'evento `onfocus` con l'attributo `autofocus`.

## Exploit

```txt
[link=" autofocus="true" onfocus="alert(1)]test[/link]
```
