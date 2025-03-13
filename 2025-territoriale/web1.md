# OliCyber.IT 2025 - Selezione territoriale

## [web] La pazienza è la virtù dei forti (462 risoluzioni)

Volevo iniziare dalla web facile, ma sembra che io debba aspettare per poterla risolvere...

Site: [http://pazienza.challs.olicyber.it](http://pazienza.challs.olicyber.it)

Autore: Lorenzo Leonardini <@pianka>

## Panoramica

La challenge si presenta come una pagina singola, che ci comunica che riceveremo la nostra flag in un futuro molto lontano.

## Soluzione

Se analizziamo i cookie della pagina, ci rendiamo conto che il sito imposta un cookie, chiamato `Get-Flag-Time`, il cui valore corrisponde al timestamp Unix al quale otterremo la flag.
Se proviamo a impostare il cookie a un timestamp passato, per esempio 0, la pagina ci ritorna la flag.
