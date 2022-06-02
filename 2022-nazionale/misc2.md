# OliCyber.IT 2022 - Competizione nazionale

## [misc-2] GGOL2 (2 risoluzioni)

```
Una dimensione in meno significa che è più facile giusto?

Nota: se trovi più di una soluzione prendi la più grande in ordine lessicografico
```

La challenge è una implementazione di [Rule 110](https://en.wikipedia.org/wiki/Rule_110): lo stato iniziale è dato dai bit della flag e viene dato in output il risultato dopo uno step di rule 100.

### Soluzione

**La prima idea** che viene in mente per risolverla è la seguente:

- per ogni bit dello stato in uscita si possono trovare tutte le disposizioni di triplette di bit che lo generano
- una volta fissate le triplette per il primo bit quelle per il secondo sono limitate perche’ devono avere un overlap di 2 bit con il pattern precedente
- la stessa cosa vale per il terzo bit e così via

**La seconda idea**, che è più semplice da scrivere, è fare un bruteforce incrementale: si prova un carattere alla volta della flag prendendo solo i caratteri che generano uno stato corrispondente al nostro target

```
guess |  ascii	   		|	uscita di rule 110
“a” -> 	01100001 ->			11100011
“r” -> 	01110010 ->			11010110
“ru” -> 0111001001110101 ->	1101011011011111
target = 					1101011011011111111...
```

Si vede che la "a" come primo carattere non va bene perchè l'uscita di uno step di rule 110 non combacia con il target, la "r" invece va bene, e così via.

### Exploit

**Exploit per la prima idea**:

```py
from collections import defaultdict
from Crypto.Util.number import long_to_bytes


RULE = 110


class RuleX:
	def __init__(self, rule, state):
		if not all(x in "01" for x in state):
			raise ValueError("State must contain only '0' or '1'")

		self.state = state
		self.l = len(state)
		self.patterns = {f"{i:03b}": str(rule >> i & 1) for i in range(8)}

		# precalculate inverse patterns
		self.inv_patters = defaultdict(list)
		for k, v in self.patterns.items():
			self.inv_patters[v].append(k)

	def step(self):
		next_state = ""
		for i in range(self.l):
			next_state += self.patterns[
				self.state[(i - 1) % self.l]
				+ self.state[i]
				+ self.state[(i + 1) % self.l]
			]
		self.state = next_state

	def step_backwards(self):
		prev_states = [""]
		for i, bit in enumerate(self.state):
			tmp = []
			for prev_state in prev_states:

				# first bit: no constraints here, all inverse patterns are valid
				if i == 0:
					tmp.extend(p[1:3] for p in self.inv_patters[bit])
					continue

				# last bit: check if it wraps around correctly and return prev_state
				if i == self.l - 1:
					if (
						self.patterns[prev_state[-2:] + prev_state[0]] == bit
						and self.patterns[prev_state[-1] + prev_state[:2]] == self.state[0]
					):
						yield prev_state
					continue

				# middle bits: check is pattern is valid against left neighbour and add bit
				for pattern in self.inv_patters[bit]:
					if pattern[0:2] == prev_state[i - 1 : i + 1]:
						tmp.append(prev_state + pattern[2])
			prev_states = tmp

state = "110101101101111111111100111011110111001101110011011100001111000111111011011111111111000111100111011100000111000011111100"

rule110 = RuleX(RULE, state)

flags = []
for s in rule110.step_backwards():
	try:
		flag = long_to_bytes(int(s, 2)).decode("ascii")
		flags.append(flag)
	except ValueError:
		pass

print(f"flag{{{max(flags)}}}")
```

**Exploit per la seconda idea**:

```py
import string


state = "110101101101111111111100111011110111001101110011011100001111000111111011011111111111000111100111011100000111000011111100"
RULE = 110


class RuleX:
    def __init__(self, rule, state):
        if not all(x in "01" for x in state):
            raise ValueError("State must contain only '0' or '1'")

        self.state = state
        self.patterns = {f"{i:03b}": str(rule >> i & 1) for i in range(8)}

    def step(self):
        next_state = ""
        l = len(self.state)
        for i in range(l):
            next_state += self.patterns[self.state[(i - 1) % l] + self.state[i] + self.state[(i + 1) % l]]
        self.state = next_state


# find middle bits
flags = [b""]
for i in range(len(state) // 8):
    tmp = []
    for flag in flags:
        for c in string.printable:
            r = RuleX(RULE, f"{int.from_bytes(flag + c.encode(), 'big'):0{(i+1)*8}b}")
            r.step()
            if r.state[1 : (i + 1) * 8 - 1] == state[1 : (i + 1) * 8 - 1]:
                tmp.append(flag + c.encode())
    flags = tmp

# enforce constraint of first and last bit (wrap around)
tmp = []
for flag in flags:
    r = RuleX(RULE, f"{int.from_bytes(flag, 'big'):0{len(flag)*8}b}")
    r.step()
    if r.state == state:
        tmp.append(flag)
flags = tmp

print(flags)
print(f"flag{{{max(flags)}}}")
```
