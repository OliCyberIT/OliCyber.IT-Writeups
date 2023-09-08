# OliCyber.IT 2023 - Competizione Nazionale

## [binary] You spin me Belandi (37 risoluzioni)

Questa challenge è di riscaldamento. Viene presentato un singolo file binario linkato dinamicamente, stranamente pesante, grande circa 8MB. Aprendo il binario con ghidra, per esempio, si nota che una immagine gif è embeddata all'interno dell'eseguibile. L'immagine è un gabibbo arrabbiato che ruota su se stesso. Uno dei frame contiene la flag, scritta in chiaro sull'immagine.

### Soluzione

Il metodo della nonna per risolvere la challenge è di fare uno screenshot mentre ghidra è aperto. Un metodo più ragionevole è per esempio di:

- estrarre la GIF dal binario, per esempio utilizzando `binwalk`
- estrarre i vari frame dalla GIF, per esempio con `gm convert gabibbo.gif -coalesce +adjoin gabibbo_Frame%3d.png`
- una volta ottenuti i vari frame si può notare che al frame numero 98 c'è la flag. Copiare manualmente sarà sicuramente più efficace di qualsiasi OCR.
