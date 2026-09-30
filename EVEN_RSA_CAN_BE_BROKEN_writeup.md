# Even RSA Can Be Broken

## Challenge Overview

The challenge gives us an RSA-encrypted flag. When we connect to the
server, it provides three values:

-   `N` --- the RSA modulus
-   `e` --- the public exponent
-   `ciphertext` --- the encrypted flag

An important observation is that reconnecting to the server gives us
different values of `N` and `ciphertext`, while `e` remains `65537`.

The challenge also provides the source code used to generate the RSA
keys and encrypt the flag.

------------------------------------------------------------------------

## Provided Source Code

``` python
from sys import exit
from Crypto.Util.number import bytes_to_long, inverse
from setup import get_primes

e = 65537

def gen_key(k):
    # Generates RSA key with k bits
    p, q = get_primes(k // 2)
    N = p * q
    d = inverse(e, (p - 1) * (q - 1))

    return ((N, e), d)

def encrypt(pubkey, m):
    N, e = pubkey
    return pow(bytes_to_long(m.encode('utf-8')), e, N)

def main(flag):
    pubkey, _privkey = gen_key(1024)
    encrypted = encrypt(pubkey, flag)
    return (pubkey[0], encrypted)

if __name__ == "__main__":
    flag = open('flag.txt', 'r').read()
    flag = flag.strip()
    N, cypher = main(flag)
    print("N:", N)
    print("e:", e)
    print("cyphertext:", cypher)
    exit()
```

From the key-generation function, we know that:

``` python
N = p * q
```

where `p` and `q` are supposed to be prime numbers.

------------------------------------------------------------------------

## Finding the Vulnerability

Normally, recovering `p` and `q` from a sufficiently large RSA modulus
`N` is computationally difficult. Therefore, the first thing to look for
is some weakness or pattern in the generated values.

After connecting to the server multiple times and comparing the
different values of `N`, one pattern stands out:

> **Every generated value of `N` is even.**

Since

`N = p * q`

an even `N` means that at least one of its factors must be even.

However, `p` and `q` are supposed to be prime numbers. The **only even
prime number is `2`**.

Therefore, one of the RSA primes must be:

``` text
q = 2
```

and the other prime can immediately be recovered using:

``` python
p = N // q
```

or simply:

``` python
p = N // 2
```

This completely breaks the RSA key because we can now reconstruct the
private exponent.

------------------------------------------------------------------------

## Example Values

For one connection to the server, we receive:

``` text
N = 23514578972536520772614351696932480279417278970800855359906493432003874544200171179759625377971984307872863405434235276639163970776527562931744347246055202

e = 65537

ciphertext = 21877807527561038280677353791710280092602954653606394902215635188806509712987515436936454487598490274894892463003970805727995777431525000107028659097796399
```

Because `N` is even:

``` python
q = 2
p = N // q
```

------------------------------------------------------------------------

## Recovering the Private Key

RSA uses Euler's totient:

`phi(N) = (p - 1)(q - 1)`

The private exponent `d` is the modular inverse of `e` modulo `phi(N)`.

Since we now know both `p` and `q`, we can calculate:

``` python
phi = (p - 1) * (q - 1)
d = inverse(e, phi)
```

Once `d` is known, RSA decryption can be performed with:

``` python
m = pow(ciphertext, d, N)
```

The result `m` is an integer, so we convert it back into bytes using
`long_to_bytes()`.

------------------------------------------------------------------------

## Exploit Script

``` python
from Crypto.Util.number import long_to_bytes, inverse

N = 23514578972536520772614351696932480279417278970800855359906493432003874544200171179759625377971984307872863405434235276639163970776527562931744347246055202

e = 65537

ciphertext = 21877807527561038280677353791710280092602954653606394902215635188806509712987515436936454487598490274894892463003970805727995777431525000107028659097796399

q = 2
p = N // q

phi = (p - 1) * (q - 1)

d = inverse(e, phi)
m = pow(ciphertext, d, N)

flag = long_to_bytes(m).decode()
print(flag)
```

### Why `long_to_bytes()`?

The encryption code first converts the flag from bytes into an integer
using:

``` python
bytes_to_long(...)
```

After RSA decryption, `m` is therefore still an integer. To recover the
original text, we perform the reverse conversion:

``` python
long_to_bytes(m)
```

and then use `.decode()` to convert the resulting bytes into a Python
string.

------------------------------------------------------------------------

## Flag

``` text
academy{tw0_1$_pr!m3cfb893a9}
```

------------------------------------------------------------------------

## Takeaway

The RSA mathematics itself was not the problem. The vulnerability came
from **bad prime generation**.

The key observation was that every modulus returned by the server was
even. Since an RSA modulus is the product of two primes and `2` is the
only even prime, this immediately revealed one of the factors:

``` text
q = 2
```

Once one factor of `N` is known, the other factor, Euler's totient, and
the private key can all be recovered.

**When solving RSA CTF challenges, always inspect the provided values
for unusual patterns before attempting expensive factorization.**
