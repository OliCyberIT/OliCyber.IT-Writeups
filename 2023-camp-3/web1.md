# OliCyber.IT 2023 - Simulazione training camp 3

## [web] è tutto solo una convenzione (161 risoluzioni)

Per ottenere la flag è necessario eseguire una richiesta HTTP con il metodo custom `FLAG`.

### Exploit

```python
#!/bin/python3

import requests
import os

URL = os.environ.get('URL', 'http://localhost:5000')

res = requests.request('FLAG', URL)
print(res.headers)
```
