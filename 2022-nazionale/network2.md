# OliCyber.IT 2022 - Competizione nazionale

## [network-2] SNECC 2 (22 risoluzioni)

`Oh no, una parte delle comunicazioni del server è cifrata... che peccato!`

L'allegato e il servizio sono gli stessi di `Strani numeri e comunicazioni cifrate 1`.

### Soluzione

Superata la prima fase, il server fornisce l'opzione per stampare il sorgente, da cui si scopre che la cosiddetta "comunicazione cifrata" è uno xor con una chiave fissata. Possiamo quindi recuperare le informazioni richieste:

```
Per avere una flag mandami un certificato X.509, firmato da te stesso e valido in questo momento, ma che soddisfi anche i seguenti requisiti: *********** TRASMISSIONE CORROTTA ***********
```

Innanzitutto bisogna quindi fornire un certificato X.509 self signed valido; inoltre, il server controlla anche che nel campo `CN` ci sia il valore `1337.olicyber`. Quest'ultimo pezzo di informazione non è pubblica, ma si capisce facilmente analizzando il traffico come nella challenge precedente. Ovviamente, la flag restituita nel traffico non è quella reale.

### Exploit

```python
import pyshark
from collections import defaultdict
from pwn import remote
import OpenSSL

def xor(a, b):
    return bytes([a[i]^b[i%len(b)] for i in range(len(a))])

def generate_cert(emailAddress="gabibbo@olicyber",
                    commonName="1337.olicyber",
                    countryName="IT",
                    localityName="ILO",
                    stateOrProvinceName="Piemonte",
                    organizationName="Olicyber",
                    organizationUnitName="repartoGabibbi",
                    serialNumber=0,
                    validityStartInSeconds=0,
                    validityEndInSeconds=10*365*24*60*60,
                    KEY_FILE = "private.key",
                    CERT_FILE="selfsigned.crt"):
    k = OpenSSL.crypto.PKey()
    k.generate_key(OpenSSL.crypto.TYPE_RSA, 4096)
    cert = OpenSSL.crypto.X509()
    cert.get_subject().C = countryName
    cert.get_subject().ST = stateOrProvinceName
    cert.get_subject().L = localityName
    cert.get_subject().O = organizationName
    cert.get_subject().OU = organizationUnitName
    cert.get_subject().CN = commonName
    cert.get_subject().emailAddress = emailAddress
    cert.set_serial_number(serialNumber)
    cert.gmtime_adj_notBefore(0)
    cert.gmtime_adj_notAfter(validityEndInSeconds)
    cert.set_issuer(cert.get_subject())
    cert.set_pubkey(k)
    cert.sign(k, 'sha512')
    return OpenSSL.crypto.dump_certificate(OpenSSL.crypto.FILETYPE_PEM, cert).decode("utf-8")

challenge_response = {}
streams = defaultdict(lambda: b"")
cap = pyshark.FileCapture("attachments/traffic.pcap")

for p in cap:
    try:
        i = int(p.tcp.stream)
        streams[i] += bytes.fromhex(p.data.data)
    except:
        pass
cap.close()

for i in streams:
    stream = streams[i]
    if b"1. Login con password" in stream:
        challenge = int(stream.split(b"\n")[0].decode())
        response = int(stream.split(b"\n")[1].decode())
        challenge_response[challenge] = response

r = remote('snecc.challs.olicyber.it', 12310)
c = int(r.recvline().decode())
r.sendline(str(challenge_response[c]).encode())

r.recvuntil(b'> ')
r.sendline(b'4')
source = r.recvuntil(b'"__main__"')
key = source.split(b'k = b"')[1].split(b'"')[0]

r.recvuntil(b'> ')
r.sendline(b'2')
r.recvuntil(b'> ')
r.sendline(xor(generate_cert().encode(), key).hex().encode())
print(xor(bytes.fromhex(r.recvline(False).decode()), key))

```
