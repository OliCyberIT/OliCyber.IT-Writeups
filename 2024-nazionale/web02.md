# OliCyber.IT 2024 - Finale Nazionale

## [web] !phishing (13 risoluzioni)

Ho creato questo fantastico sito che non fa assolutamente niente (a meno che tu non sia un admin :P).
Accetta solo registrazioni usando una mail `@fakemail.olicyber.it`, fortunatamente ne puoi richiedere una anche tu!

Nota: l'amministratore del sito ha seguito un training contro il phishing, quindi cliccherà solo su link proveniente dalla mail ufficiale della piattaforma!

Nota: non perdere tempo sul servizio fakemail, è stato creato per simulare le mail, non è il fulcro della challenge.

Challenge: [http://not-phishing.challs.olicyber.it:38100](http://not-phishing.challs.olicyber.it:38100)

Fakemail: [http://not-phishing-fakemail.challs.olicyber.it:38101](http://not-phishing-fakemail.challs.olicyber.it:38101)

Author: Aleandro Prudenzano <@drw0if>

## Panoramica

La challenge appare come un sito web che offre funzionalità apparentemente limitate, ma consente:
- Registrazione
- Login
- Verifica dell'email
- Login senza password se si ha accesso alla email associata
- Accesso a un portale admin che fornisce la flag se si hanno i privilegi necessari

Gli aspetti cruciali riguardano l'uso dell'email per la verifica dell'account e il login senza password.

Le email inviate non sono reali ma possono essere visualizzate usando il portale `fakemail` fornito. Gli utenti possono registrarsi su questo portale, effettuare il login e controllare la propria casella di posta.

Questo portale consente solo di ricevere e cancellare email, non di inviarle.

## Vulnerabilità

Il servizio ha un nginx mal configurato, rendendolo vulnerabile a spoofing dell'header `Host`. L'unico host virtuale configurato (attraverso il blocco `server`) è il servizio stesso.

Essendo il primo e unico server configurato, viene utilizzato come predefinito per gestire ogni richiesta che arriva alla porta nginx, senza effettuare alcun controllo sul valore dell'header `Host`.

Per costruire le email, l'applicazione utilizza il contenuto dell'header `Host`:

```php
$domain_name = $_SERVER['HTTP_HOST'];
send_mail(
    $email,
    "Login",
    "Go to http://$domain_name/token_login.php?token=$token to log in!"
);
```
## Soluzione

Dal momento che possiamo modificare liberamente il contenuto dell'header `Host` e questo viene utilizzato per costruire il dominio del link di login, possiamo richiedere il login dell'account admin specificando, come host, un servizio controllato da noi.

In questo modo, quando l'amministratore visita il link nell'email del sito ufficiale (fidato), possiamo catturare il token di accesso.

Una volta ottenuta una sessione come admin, basta andare alla pagina `/admin.php` per ottenere la flag.
