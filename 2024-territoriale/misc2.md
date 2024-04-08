# OliCyber.IT 2024 - Selezione territoriale

## [misc] Easy login (282 risoluzioni)

Siamo riusciti a intercettare un utente durante un login, prova a trovare la flag.

Sito: [http://easylogin.challs.olicyber.it](http://easylogin.challs.olicyber.it)

Autore: Giovanni Minotti <@Giotino>

## Soluzione

Analizzando il traffico nel file PCAP (oer esempio usando Wireshark), è facile trovare i campi `username` e `password` utilizzati per il login. Il campo `TOTP`, però, è scaduto e non può essere generato perchè l'utente non possiede il segreto necessario per farlo. Il PCAP, però, contiene anche il cookie `session`, che è ancora valido e può essere utilizzato per effettuare il login.
