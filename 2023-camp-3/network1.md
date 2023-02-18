# OliCyber.IT 2023 - Simulazione training camp 3

## [network] Furto di dati (2 risoluzioni)

La challenge consiste in un file pcap contenente log del traffico di rete di un'azienda.

### Soluzione

Il file pcap è intenzionalmente molto grosso, a emulare un contesto più o meno realistico.
Nella descrizione si cita l'assenza di informazioni nei pacchetti TCP e UDP. Impostando come filtro wireshark

```sh
!tcp && !udp
```

si ottiene una lista lunga di pacchetti ARP.
Analizzandoli si nota che sono tutti uguali a meno dell'indirizzo MAC sorgente.
Escludendo le richieste ARP legittime (quelle con indirizzo sorgente!=1.3.3.7, chiaramente fittizio), concatenando tutti gli indirizzi MAC, togliendo i ":", unhexlify, base64 decode, salvato su file si trova un immagine che contiene la flag.

### Exploit

```bash
tshark -r furto.pcapng -Y 'arp.src.proto_ipv4==1.3.3.7' -T fields -e arp.src.hw_mac | tr -d '\n:' | xxd -r -p | base64 -d > image.png
```
