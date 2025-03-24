# Reverse Engineering Instagram

## Password Encryption (pwd_encoder.js)
### Parameters
- Instagram Encryption Key ID
- Instagram Encryption Public Key
- Password to encrypt
- Concurrent timestamp (with 10 digits)
- Instagram Encryption Version (10 for encryption)

**Nodejs execute**
```sh
node pwd_encoder.js 158 '425cffed5f79336b65c68c19da3e72b1c08545253057db6dce39885f7dca1c6e' 'your-password-here' '1742679434' 10
```
**Python**
```python
import time
import subprocess

result = subprocess.run([
    'node',
    'pwd_encoder.js',
    '158',
    '425cffed5f79336b65c68c19da3e72b1c08545253057db6dce39885f7dca1c6e',
    'PASSWORD HERE',
    str(round(time.time())),
    '10'
], capture_output=True, text=True)

print(result.stdout.strip())
```
It will result in the password:
```
#PWD_INSTAGRAM_BROWSER:10:1742679434:AZ1QAC1dEJegrbNAYcDn7aPPzXcnIfO5x2mhi9Ad0Ax45eYKn45W88XlhGm95iwIt10Y5bvdd+ceEjSj4etqaILHLpraxojNY4nIn13Sdggc7oYjv5y5n/9KIzrNgThBBZ9BxTEN7r1ZuWhXrOd6p4yvKbT8dQ==
```
***Note*** The keyid and publickey are not static, change from time to time. They are obtained by scraping the HTML of the Instagram page.


## Instagram Headers/Payload Values (reversed.py)
### X-Mid Header Generation
```python
import math
import random
import functools

def random_uint32():
    return math.floor(random.random() * 4294967296)

def to_string(n):
    chars = '0123456789abcdefghijklmnopqrstuvwxyz'
    result = ''
    while n:
        n, r = divmod(n, 36)
        result = chars[r] + result
    return result or '0'

def machine_id():
    return functools.reduce(
        lambda a, _: a + to_string(random_uint32()),
        [0] * 8,
        ''
    )
```
```python
print(machine_id())
# result:
# 1olgdc41oa6xbd5hetz5agulv79tei931on7kdg1i9jnzz1pgez3y
```

### X-Web-Session-ID
```python
import math
import random

def to_string(n):
    chars = '0123456789abcdefghijklmnopqrstuvwxyz'
    result = ''
    while n:
        n, r = divmod(n, 36)
        result = chars[r] + result
    return result or '0'

def web_session_id(extra=False, c=None):
    def _p(j=6):
        a = math.floor(random.random() * 2176782336)
        a = to_string(a)
        return '0' * (j - len(a)) + a

    if extra:
        a = _p()
        b = _p()
    else:
        a = '' #  webstorage
        b = '' #  webstorage
    if c is None:
        c = _p()
    return a + ':' + b + ':' + c
```
```python
sid = web_session_id() # when the browser has no data stored
print(sid)
# ::1p7dm3

print(web_session_id(extra=True, c=sid.split(':')[2]))
# lejnny:v14jyj:1p7dm3
```

### Jazoest Payload Parameter
- This value takes a string and represents it as a number

```python
def get_numeric_value(string):
    c = 0
    sprinkle_version = '2'

    for x in range(len(string)):
        c += ord(string[x])
    return sprinkle_version + str(c)
```
```python
jazoest = get_numeric_value('csrf_token') # for signup

print(jazoest)
# result:
# 21070
```

