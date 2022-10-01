# OliCyber.IT 2021 - Competizione nazionale

## [crypto-1] Un po' di storia (69 risoluzioni)

`Abbiamo ritrovato questa pergamena del XVII secolo, che è stata cifrata con un cifrario del 1586; attenzione alle varianti!`

Il testo cifrato è `kwewgguwztzvackcdxmbltjyjppdfvwwvgfhrmrxmkgjaewyjmjamqgjmrwmgjrxtziklbrulelxzqsssyfgyyhfihmxkbrfpxrvblflbpwkzlfcahsqawgpgfaxieikmuxefhlzzxmepmxoxjvmznplnimfygtgxfvfsdigmyowvsqmftjbrhhkgizvkvprzvallxhqxlmcplyamzdwislpayssqpbanikxlslcbdyureuavnhklrawqakwmedmnfquhrrcffvhsgjcdmpxmowxspyhafdgaxsmzcdbxshuvfjehvqlnllmhfngvwmhvpbfhwzjnsjsrjsrqwuhradaryrhruoyorvxyerdnrrztxihjuaiyqazdqglwjgicqfinvfmwubpksfmxurcbqmiaweefayeoqrmznnletoagjiyzezqvvmvikbbuklwkjwvvazjpwvzwrdymawfhpnhxogulxksrewzhqssjwzhavbtxhrjkhzi`.

### Soluzione

Da una rapida ricerca si scopre che il cifrario del 1586 è il cifrario di [Vigenère](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher). La versione base non sembra però essere quella corretta, infatti la descrizione ci suggerisce di focalizzarci sulle varianti.

Tra le 4 varianti elencate nell'articolo proposto, la variante `autokey` è quella che fornisce la flag, ad esempio usando un tool automatico come [dcode](https://www.dcode.fr/autoclave-cipher).

A questo punto, rimane soltanto da convertire il testo decifrato nel formato della flag.
