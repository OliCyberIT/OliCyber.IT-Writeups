# OliCyber.IT 2023 - Competizione Nazionale

## [network] DoNotReverseMe (X risoluzioni)

La challenge fornisce un client eseguibile che si connette ad un endpoint remoto, scambiando dei dati ("autenticando" il client in quanto le operazioni effettuate su questi dati vengono controllate) e stampando poi parte della flag.

Il binario è volutamente difficile da reversare, in quanto non è quello l'obiettivo della challenge.

### Soluzione

Per recuperare la flag per intero, è sufficiente intercettare il traffico scambiato tra il client e il server, ad esempio con tcpdump da linea di comando o con Wireshark da interfaccia grafica.

### Exploit

```bash
tcpdump -i any -w out.pcap
```
