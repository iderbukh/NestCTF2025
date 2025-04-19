e = 3 
its vulnerable to Low Exponent Attack
google Low Exponent attack

solve.py 
```
import gmpy2

c = int(input("c: "))
n = int(input("n: "))
e = int(input("e: "))

gs = gmpy2.mpz(c)
gm = gmpy2.mpz(n)
ge = gmpy2.mpz(e)

root, exact = gmpy2.iroot(gs, ge)

if(exact):
  print(bytes.fromhex(hex(int(root))[2:]))
```

flag: nest{wow1337}