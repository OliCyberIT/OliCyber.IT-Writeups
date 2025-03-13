# OliCyber.IT 2025 - Selezione territoriale

## [rev] Baity5 (389 risoluzioni)

Mi hanno dato questo eseguibile per decodificare una flag in Baity5... ma non stampa niente :((((( aiuto!

Autore: Giulia Martino <@Giulia>

## Panoramica

Il binario in allegato contiene una stringa, chiamata `enc_flag`, che rappresenta la codifica in Base85 della flag. E' possibile dedurre che si stia utilizzando Base85 sia dal nome della challenge stessa, sia dalla funzione chiamata dentro al binario, `decode_base85`. Il binario però non stampa la flag decodificata.

## Soluzione

Una volta colto che si tratta di Base85, è sufficiente estrarre la stringa dal binario (per esempio con il comando `strings` o utilizzando Ghidra), e decodificarla per ottenere la flag. Per decodificarla è possibile utilizzare un tool online come CyberChef, oppure il modulo builtin di Python `base64`, che gestisce anche Base85.

## Exploit

```python
import base64

enc_flag = "ENCODED_FLAG_HERE"
print(base64.a85decode(enc_flag).decode())
```
