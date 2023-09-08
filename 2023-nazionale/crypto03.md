# OliCyber.IT 2023 - Finale Nazionale

## [crypto] Ritorno al passato (0 risoluzioni)

Il server permette di registrare utenti tramite username e password. Inoltre ad ogni utente viene assegnato un token di recupero password.

In particolare, il token viene usato come chiave DES per cifrare la password dell'utente nella funzione `recupera_password`, in modalità CBC.

Inoltre c'è una funziona che controlla la validità di una password cifrata, provando a decifrarla.

### Soluzione

La vulnerabilità sta nel fatto che la funzione `recupera_password` fa `usr.strip()` per recuperare il token dal database. Questo vuol dire che abbiamo un modo di calcolare `Encrypt('{"usr": "admin    ", "psw": "admin_password"}')` per un numero arbitrario di spazi.

Inoltre, la funzione `inutile` è una sorta di padding oracle, dato che controlla `"admin_password" in Decrypt(token)`.

Con queste due funzioni possiamo creare un partial decryption oracle, in particolare una funzione che dato un blocco arbitrario calcola il primo carattere della decryption DES di quel blocco.

Possiamo infatti calcolare infinite encryption del seguente dizionario:

`{"usr": |"admin  |     ", |"pwd": "|pwd_block1|pwd_block2|"}`

Sostituendo l'ultimo blocco di cifrato, corrispondente alla cifratura di `"}`, con un qualunque blocco, la decifratura conterrà il carattere `Decrypt(block) ^ ct[48]`, e la funzione `inutile` ci avviserà di quando questo carattere è una virgoletta.

In particolare, creando un dizionario contenente valide cifrature per ogni possibile valore di `ct[48]`, potremo decifrare il primo carattere di blocchi arbitrari.

Usando una diversa quantità di spazi per lo username, possiamo spostare ogni carattere della password all'inizio di un blocco, e poi decifrarlo.

### Exploit

```python
# Allineiamo lo user per avere la virgoletta singola al fondo
# {... || "admin-- || -----",- || "psw":-" || .... || .... || "}
admin_username = "admin" + " "*7

# Esegue recupera_password
def query_recovery(name):
    global r
    r.recvuntil(b"> ")
    r.sendline(b"3")
    r.recvuntil(b": ")
    r.sendline(name)
    res = bytes.fromhex(r.recvline(False).split()[-1].decode())
    return res

# Esegue inutile sull'admin
def query_check(token):
    global r
    r.recvuntil(b"> ")
    r.sendline(b"4")
    r.recvuntil(b": ")
    r.sendline(b"admin")
    r.recvuntil(b": ")
    r.sendline(token.hex().encode())
    res = r.recvline()
    return b"proprio funzionare!" in res

# Troviamo dei ciphertext validi per ogni possibile carattere in posizione 48
def create_dict():
    d = {}
    while len(d) < 256:
        res = query_recovery(admin_username)
        d[res[48]] = res
    return d

# Questa funzione restituisce il primo carattere di Decrypt(block), utilizzando il decryption oracle di inutile
def char_oracle(block, d):
    for i in range(256):
        token = d[i][0:56] + block
        if query_check(token):
            return i ^ ord('"')

def get_byte(i,d):
    enc = query_recovery(admin_username + " " * (8 - i % 8))
    idx = 6 + (i//8) # 4 blocchi inutili + 2 blocchi di psw
    block = enc[8*idx : 8*(idx + 1)]
    b = char_oracle(block, d)
    assert b != None
    return b ^ enc[8*(idx - 1)] # Decifriamo il CBC mode


r = remote(HOST, PORT)
d = create_dict()
rec = [get_byte(i, d) for i in range(16)]

r.recvuntil(b"> ")
r.sendline(b"2")
r.recvuntil(b": ")
r.sendline(b"admin")
r.recvuntil(b": ")
r.sendline(bytes(rec))
r.recvline()
print(r.recvline())
```
