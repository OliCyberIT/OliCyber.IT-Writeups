# OliCyber.IT 2024 - Finale Nazionale

## [web] AffiliatedStore (8 risoluzioni)

Stiamo lavorando a questo nuovo negozio online con un programma di affiliazione complesso. Siamo ancora al lavoro e la parte dei pagamenti non è ancora completa. Nel frattempo, puoi aiutarci a testare questa parte per vulnerabilità?

Sito: [http://affiliatedstore.challs.olicyber.it](http://affiliatedstore.challs.olicyber.it)

Author: Lorenzo Leonardini <@pianka>

## Panoramica

La sfida presenta un negozio online. Durante la registrazione possiamo specificare un id di affiliazione. Se qualcuno effettua un acquisto dopo essersi registrato con il nostro id, possiamo vedere i dettagli dell'ordine nel nostro pannello di controllo.

C'è una funzionalità di feedback che consente di condividere il nostro carrello con un amministratore. Lo stato del carrello viene memorizzato nei parametri dell'URL. Quando inviamo un carrello per il feedback, l'amministratore inserisce la flag nel messaggio dell'ordine e procede con l'acquisto.

## Soluzione

Il nostro obiettivo è leggere la flag digitata nel messaggio personalizzato dell'ordine. Se ti registri con un id di affiliazione, tutti i tuoi messaggi personalizzati diventano disponibili all'utente che ti ha fornito l'id di affiliazione. Quindi vogliamo che l'amministratore abbia il nostro id di affiliazione.

Curiosamente, l'id di affiliazione è memorizzato in sessionStorage.

La vulnerabilità risiede nella pagina del carrello, dove il seguente codice è vulnerabile a prototype pollution:

```js
const cart = JSON.parse(atob(new URL(location.href).searchParams.get('cart')));

const products = {};

cart.forEach((el) => {
	const product = products[el.id] || (products[el.id] = {});

	for (const [key, value] of Object.entries(el)) {
		if (key === 'id') continue;
		product[key] = DOMPurify.sanitize(value);
	}
});
```

se aggiungiamo al carrello un elemento con il seguente formato:

```json
{
	"id": "__proto__",
	"affiliation": "[our id]"
}
```

possiamo utilizzare la prototype pollution per leggere un valore a piacere da sessionStorage:

```js
fetch('/api/order', {
	method: 'POST',
	headers: {
		'Content-Type': 'application/json'
	},
	body: JSON.stringify({
		cart: cart,
		message: customMessage.value,
		affiliation: sessionStorage.affiliation
	})
});
```
