# OliCyber.IT 2024 - Selezione territoriale

## [web] Pretty please (521 risoluzioni)

A volte basta chiedere :)

Sito: [http://prettyplease.challs.olicyber.it](http://prettyplease.challs.olicyber.it)

Autore: Stefano Alberto <@Xato>

## Panoramica

La challenge è una semplice pagina web che permette di chiedere la flag con un form a scelta multipla.

Leggendo il sorgente (disponibile direttamente nella pagina) si può vedere come viene gestito il form inviato al server.

```php
if (isset($_POST['how'])) {

    switch ($_POST['how']) {
        case 'now':
            echo '<div class="alert alert-danger">Please, learn some good manners</div>';
            break;
        case 'please':
            echo '<div class="alert alert-danger">Mmmmh, you can do better</div>';
            break;
        case 'gabibbo':
            echo '<div class="alert alert-danger">How do you know my name?!</div>';
            echo '<style>body{ background: url("gabibbo.jpg") fixed center; background-size: cover } form, h3 {background-color: white}</style>';
            break;
        case 'pretty please':
            include_once('secret.php');
            echo '<div class="alert alert-success">Now we are talking! ' . $FLAG . '</div>';
            break;
        default:
            echo '<div class="alert alert-danger">I don\'t understand you...</div>';
            break;
    }
}
```

Tutte le opzioni disponibili dal menù di selezione non ci permettono di ottenere la flag; per farlo dobbiamo forzare l'invio dell'opzione "pretty please".

## Soluzione

Per inviare un valore arbitrario si può procedere in due modi:

- Modificare la pagina nel browser usando gli strumenti sviluppatore (usando il tasto F12) per aggiungere un opzione con valore "pretty please" nel menù.
- Forgiare la richiesta HTTP direttamente per inviare l'opzione arbitraria.

Si riporta il codice Python che permette di effettuare la richiesta necessaria per ottenere la flag e stamparla.

```python
URL = "http://prettyplease.challs.olicyber.it"
r = requests.post(URL, data={'how': 'pretty please'})
flag = re.search(r'flag{.*}', r.text).group(0)
print(flag)
```
