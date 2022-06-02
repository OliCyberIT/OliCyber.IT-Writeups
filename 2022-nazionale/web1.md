# OliCyber.IT 2022 - Competizione nazionale

## [web-1] BibboDB (32 risoluzioni)

La challenge consiste in un sito web che raccoglie informazioni riguardo le produzioni artistiche del Gabibbo.

Le informazioni si possono essere filtrate per tipologia o anno di produzione, ma alcune di queste scelte sono vietate.

### Soluzione

La flag si trova nella categoria "secret", ma vistitando http://bibbodb.challs.olicyber.it/type?filter=secret viene restituito un errore al posto della flag.

Il servizio usa mongodb, un database NoSQL, per filtrare i dati da mostrare all'utente.

Questo filtro è vulnerabile a NoSQL injection, permettendo di modificare la query sfruttando la sintassi di mongodb.

In particolare, per ottenere la flag, basta inviare come filtro un oggetto simile a `{"$eq":"secret"}`.
Questo oggetto passa il filtro e richiede al database tutti i dati della tipologia "secret".

### Exploit

Per inviare l'oggetto al server e ottenere la flag basta visitare il link http://bibbodb.challs.olicyber.it/type?filter[$eq]=secret
