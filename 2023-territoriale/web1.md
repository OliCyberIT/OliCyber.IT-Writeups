# OliCyber.IT 2023 - Selezione territoriale

## [web-1] Gabibbo's treasure (550 risoluzioni)

La challenge implementa un semplice controllo delle credenziali client-side

### Soluzione

Analizzando il sorgente della pagina visitata trovaiamo lo script JavaScript che viene eseguito al click del pulsante.

```js
const password = document.getElementById("password").value;
if (password !== "Belandi, dammi la flag!") {
  document.getElementById("result").innerText = "Password errata";
} else {
  flag = await (
    await fetch("/flag?password=" + encodeURIComponent(password))
  ).text();
  document.getElementById("result").innerText = flag;
}
```

Da qui possiamo ottenere la password `Belandi, dammi la flag!` che ci permetterà di ricevere la flag.

### Exploit

```python
import requests

r = requests.get('http://treasure.challs.olicyber.it/flag?password=Belandi%2C%20dammi%20la%20flag!')
print(r.text)
```
