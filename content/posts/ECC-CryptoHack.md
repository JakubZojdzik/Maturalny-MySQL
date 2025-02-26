---
title: 'ECC CryptoHack'
summary: ''
date: 2024-06-22T21:36:19+02:00
draft: false
---

This post shows solutions for all challenges from the [CryptoHack elliptic curve course](https://cryptohack.org/courses/elliptic) with brief explanation. I hope you will find it helpful. You can find these solution with all challenge files on my [github](https://github.com/JakubZojdzik/ECC-CryptoHack). Feel free to open pull request if you can correct something.

## Challenges list

1. [Background Reading](#background-reading)
2. [Point Negation](#point-negation)
3. [Point Addition](#point-addition)
4. [Scalar Multiplication](#scalar-multiplication)
5. [Curves and Logs](#curves-and-logs)
6. [Efficient Exchange](#efficient-exchange)
7. [Montgomery's Ladder](#montgomerys-ladder)
8. [Smooth Criminal](#smooth-criminal)
9. [Curveball](#curveball)
10. [ProSign 3](#prosign-3)
11. [Moving Problems](#moving-problems)

## Background Reading

> The flag is the name we give groups with a commutative operation.

These are [abelian groups](https://en.wikipedia.org/wiki/Abelian_group)

Flag: `crypto{abelian}`

---

## Point Negation

> For all the challenges in the starter set, we will be working with the elliptic curve  
> `E: Y2 = X3 + 497 X + 1768, p: 9739`  
> Using the above curve, and the point `P(8045,6936)`, find the point `Q(x,y)` such that `P + Q = O`.

If `P + Q = 0`, then `P = -Q`. Negation of a point over elliptic curve is its symmetry around axis `X`. Then, we also need to ensure that resulting point is over a finite field `mod 9739`.

```python
P = (8045, 6936)
Q = [P[0], -P[1]]
Q[1] %= 9739

print(Q)
```

Flag: `crypto{8045,2803}`

---

## Point Addition

> We will work with the following elliptic curve, and prime:  
> `E: Y2 = X3 + 497 X + 1768, p: 9739`  
> You can test your algorithm by asserting: `X + Y = (1024, 4440)` and `X + X = (7284, 2107)` for `X = (5274, 2841)` and `Y = (8669, 740)`.  
> Using the above curve, and the points `P = (493, 5564)`, `Q = (1539, 4742)`, `R = (4403,5202)`, find the point `S(x,y) = P + P + Q + R` by implementing the above algorithm.

It is just prescribing provided algorithm

```python
a = 497
b = 1768
p = 9739

def add_points(P, Q):
    if(P == (0, 0)):
        return Q
    if(Q == (0, 0)):
        return P

    x1, y1 = P
    x2, y2 = Q
    if(x1 == x2 and y1 == -y2):
        return (0, 0)

    if(P != Q):
        lamb = (y2 - y1) * pow((x2 - x1), -1, p)
    if(P == Q):
        lamb = (3*x1**2 + a) * pow(2*y1, -1, p)
    x3 = lamb**2 - x1 - x2
    y3 = lamb * (x1-x3) - y1
    return (x3%p, y3%p)


X = (5274, 2841)
Y = (8669, 740)

assert add_points(X, Y) == (1024, 4440)
assert add_points(X, X) == (7284, 2107)

P = (493, 5564)
Q = (1539, 4742)
R = (4403,5202)

res = add_points(P, P)
res = add_points(res, Q)
res = add_points(res, R)

print(res)
```

Flag: `crypto{4215,2162}`

---

## Scalar Multiplication

> We will work with the following elliptic curve, and prime:  
> `E: Y2 = X3 + 497 X + 1768, p: 9739`  
> You can test your algorithm by asserting: `1337 X = (1089, 6931)` for `X = (5323, 5438)`.  
> Using the above curve, and the points `P = (2339, 2213)`, find the point `Q(x,y) = 7863 P` by implementing the above algorithm.  

It is again prescribing provided algorithm. Add function from previous challenge is helpful here.

```python
a = 497
b = 1768
p = 9739

# Function from previous challenge
def add_points(P, Q):
    if(P == (0, 0)):
        return Q
    if(Q == (0, 0)):
        return P

    x1, y1 = P
    x2, y2 = Q
    if(x1 == x2 and y1 == -y2):
        return (0, 0)

    if(P != Q):
        lamb = (y2 - y1) * pow((x2 - x1), -1, p)
    if(P == Q):
        lamb = (3*x1**2 + a) * pow(2*y1, -1, p)
    x3 = lamb**2 - x1 - x2
    y3 = lamb * (x1-x3) - y1
    return (x3%p, y3%p)

def multiply_points(P, n):
    Q = P
    R = (0, 0)

    while(n > 0):
        if(n % 2 == 1):
            R = add_points(R, Q)
        Q = add_points(Q, Q)
        n = n//2
    return R


X = (5323, 5438)
assert multiply_points(X, 1337) == (1089, 6931)

P = (2339, 2213)
print(multiply_points(P, 7863))
```

Flag: `crypto{9467,2742}`

---

## Curves and Logs

> Calculate the shared secret after Alice sends you `QA = (815, 3190)`, with your secret integer `nB = 1829`.  
> Generate a key by calculating the SHA1 hash of the x coordinate (take the integer representation of the coordinate and cast it to a string). The flag is the hexdigest you find.

In order to calculate shared private key, we multiply Alice's part with our secret number. We can use multiply and add functions from previous challenges.

```python
from hashlib import sha1

a = 497
b = 1768
p = 9739

# Functions from previous challenges
def add_points(P, Q):
    if(P == (0, 0)):
        return Q
    if(Q == (0, 0)):
        return P

    x1, y1 = P
    x2, y2 = Q
    if(x1 == x2 and y1 == -y2):
        return (0, 0)

    if(P != Q):
        lamb = (y2 - y1) * pow((x2 - x1), -1, p)
    if(P == Q):
        lamb = (3*x1**2 + a) * pow(2*y1, -1, p)
    x3 = lamb**2 - x1 - x2
    y3 = lamb * (x1-x3) - y1
    return (x3%p, y3%p)

def multiply_points(P, n):
    Q = P
    R = (0, 0)

    while(n > 0):
        if(n % 2 == 1):
            R = add_points(R, Q)
        Q = add_points(Q, Q)
        n = n//2
    return R

Q = (815, 3190)

P = multiply_points(Q, 1829)

flag = sha1(str(P[0]).encode()).hexdigest()
print(flag)
```

Flag: `crypto{80e5212754a824d3a4aed185ace4f9cac0f908bf}`

---

## Efficient Exchange

> Calculate the shared secret after Alice sends you `q_x = 4726`, with your secret integer `nB = 6534`.

Again, everything needed for calculating secret is given. First we calculate Y coordinate of point Q, and than we multiply it by our secret number `n`. It's X coordinate is the shared secret we were looking for.

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
import hashlib

def is_pkcs7_padded(message):
    padding = message[-message[-1]:]
    return all(padding[i] == len(padding) for i in range(0, len(padding)))

def decrypt_flag(shared_secret: int, iv: str, ciphertext: str):
    # Derive AES key from shared secret
    sha1 = hashlib.sha1()
    sha1.update(str(shared_secret).encode('ascii'))
    key = sha1.digest()[:16]
    # Decrypt flag
    ciphertext = bytes.fromhex(ciphertext)
    iv = bytes.fromhex(iv)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = cipher.decrypt(ciphertext)

    if is_pkcs7_padded(plaintext):
        return unpad(plaintext, 16).decode('ascii')
    else:
        return plaintext.decode('ascii')

# Functions from previous challenges
def add_points(P, Q):
    if(P == (0, 0)):
        return Q
    if(Q == (0, 0)):
        return P

    x1, y1 = P
    x2, y2 = Q
    if(x1 == x2 and y1 == -y2):
        return (0, 0)

    if(P != Q):
        lamb = (y2 - y1) * pow((x2 - x1), -1, p)
    if(P == Q):
        lamb = (3*x1**2 + a) * pow(2*y1, -1, p)
    x3 = lamb**2 - x1 - x2
    y3 = lamb * (x1-x3) - y1
    return (x3%p, y3%p)

def multiply_points(P, n):
    Q = P
    R = (0, 0)

    while(n > 0):
        if(n % 2 == 1):
            R = add_points(R, Q)
        Q = add_points(Q, Q)
        n = n//2
    return R

a = 497
b = 1768
p = 9739

q_x = 4726
# y**2 coordinate of point Q
q_y2 = q_x**3 + a*q_x + b
# it is so easy thanks to p ≡ 3 mod 4
q_y = pow(q_y2, ((p + 1) // 4), p)
Q = (q_x, q_y)
nB = 6534
P = multiply_points(Q, nB)

shared_secret = P[0]
iv = 'cd9da9f1c60925922377ea952afc212c'
ciphertext = 'febcbe3a3414a730b125931dccf912d2239f3e969c4334d95ed0ec86f6449ad8'

print(decrypt_flag(shared_secret, iv, ciphertext))

```

Flag: `crypto{3ff1c1ent_k3y_3xch4ng3}`

---

## Montgomery's Ladder

> find the x-coordinate (decimal representation) of point `Q = [0x1337c0decafe] G` by implementing the above algorithm.

In this challenge, Montgomery form of curve was provided, so new addition and multiplication formulas were needed. This time, we had to multiply generator point by `0x1337c0decafe`, but firstly, we had to find Y coordinate of `G`. It is easy to get square of it by just passing it's x to the curve formula, but then it would be necessary to find it's root what is more difficult. One could either implement an algorithm for finding it or use a ready function from sage math and get whole point `G` with one function `lift_x`.

```python
# sage
class Point():
    def __init__(self, x = 0, y = 0):
        self.x = y
        self.y = x

p = pow(2, 255) - 19
A = 486662
B = 1
G = Point()
G.x = 9

# Use sage to find Y coordinate of generator G
F = GF(p)
E = EllipticCurve(F, [0,A,0,B,0])
G.y = E.lift_x(G.x)[1]

def add(P: Point, Q: Point):
    m = ((Q.y - P.y) * pow(Q.x - P.x, -1, p)) % p
    R = Point()
    R.x = (B * m*m - A  - P.x - Q.x) % p
    R.y = (m * (P.x - R.x) - P.y) % p
    return R

def double_point(P: Point):
    m = ((3*P.x**2 + 2*A*P.x + 1) * pow(2*B*P.y, -1, p)) % p
    R = Point()
    R.x = (B*m**2 - A - 2*P.x) % p
    R.y = (m * (P.x - R.x) - P.y) % p
    return R

def mult(P: Point, k: int):
    R0 = P
    R1 = double_point(P)
    k_bin = bin(k)[3:]
    for bit in k_bin:
        if bit == '0':
            (R0, R1) = (double_point(R0), add(R0, R1))
        else:
            (R0, R1) = (add(R0, R1), double_point(R1))
    return R0

Q = mult(G, 0x1337c0decafe)
print(Q.x)


```

Flag: `crypto{49231350462786016064336756977412654793383964726771892982507420921563002378152}`

---

## Smooth Criminal

> Spent my morning reading up on ECC and now I'm ready to start encrypting my messages. Sent a flag to Bob today, but you'll never read it.

In this challenge curve with insecure parameters is given. It is vulnerable to Pohlig-Hellman attack, because of small prime factors of it's order. Math behind this attack is quite complicated, so I used a ready implementation and it worked :)

```python
# sage
from Crypto.Cipher import AES
import hashlib

def decrypt_flag(shared_secret: int):
    sha1 = hashlib.sha1()
    sha1.update(str(shared_secret).encode('ascii'))
    key = sha1.digest()[:16]

    iv = bytes.fromhex('07e2628b590095a5e332d397b8a59aa7')
    ciphertext = bytes.fromhex('8220b7c47b36777a737f5ef9caa2814cf20c1c1ef496ec21a9b4833da24a008d0870d3ac3a6ad80065c138a2ed6136af')
    cipher = AES.new(key, AES.MODE_CBC, iv)

    # Prepare data to send
    pt = cipher.decrypt(ciphertext)
    return pt

# Define curve
p = 310717010502520989590157367261876774703
a = 2
b = 3
F = GF(p)
EC = EllipticCurve(F, [a, b])

G = EC(179210853392303317793440285562762725654, 105268671499942631758568591033409611165)

# Q = n1 * G
Q = EC(280810182131414898730378982766101210916, 291506490768054478159835604632710368904)

# B = n2 * G
B = EC(272640099140026426377756188075937988094, 51062462309521034358726608268084433317)

# Pohlig Hellman
ordG = G.order()
factors = factor(ordG)
smallres = []
smallmod = []
for f in factors:
    smallmod.append(f[0]^f[1])
    curr = ordG // (f[0]^f[1])
    Q2 = curr * Q
    P2 = curr * G
    if(f[1] != 1):
        smallres.append(discrete_log(Q2, P2, operation='+'))
    else:
        smallres.append(discrete_log_rho(Q2, P2, operation='+'))

# Chinese Remainder Theorem
n1 = crt(smallres, smallmod)
assert n1*G == Q

# Shared secret = Q * n2 = B * n1 = G * n1 * n2
print(decrypt_flag((n1*B)[0]))
```

Flag: `crypto{n07_4ll_curv3s_4r3_s4f3_curv3s}`

---

## Curveball

> Here's my secure search engine, which will only search for hosts it has in its trusted certificate cache.

We have to find a pair `private_key` and `generator`, which product will be equal to public key assigned to `www.bing.com`. The easiest way would be to set `private_key=1`, but it isn't allowed. Natural next idea is to set `private_key=2` and try to find matching generator. I have found [this easy method](https://crypto.stackexchange.com/questions/55069/elliptic-curve-divide-by-2#answer-55109).

```python
import fastecdsa
from fastecdsa.point import Point
from pwn import *


G = fastecdsa.curve.P256.G
assert G.x, G.y == [0x6B17D1F2E12C4247F8BCE6E563A440F277037D812DEB33A0F4A13945D898C296,
                    0x4FE342E2FE1A7F9B8EE7EB4A7C0F9E162BCE33576B315ECECBB6406837BF51F5]


p = fastecdsa.curve.P256.p
q = fastecdsa.curve.P256.q
target = Point(0x3B827FF5E8EA151E6E51F8D0ABF08D90F571914A595891F9998A5BD49DFA3531, 0xAB61705C502CA0F7AA127DEC096B2BBDC9BD3B4281808B3740C320810888592A)

# https://crypto.stackexchange.com/questions/55069/elliptic-curve-divide-by-2#answer-55109
i = pow(2, -1, q)
P = target * i

conn = remote('socket.cryptohack.org', 13382)
conn.recvline()

payload = '{"host": "www.bing.pl", "private_key": 2, "generator": [' + str(P.x) + ',' + str(P.y) + '], "curve": "secp256r1"}'

conn.sendline(payload.encode())
print(conn.recvline())
```

Flag: `crypto{Curveballing_Microsoft_CVE-2020-0601}`

---

## ProSign 3

> This is my secure timestamp signing server. Only if you can produce a signature for "unlock" can you learn more.

Here, we have to sign `unlock` message with ECDSA without provided private key, but with possibility to sign some restricted messages. In ECDSA signing should be performed with cryptographically secure random integer `k` less than `n`, which is integer order of `G`. In provided algorithm, `n` variable is assigned to number of second in current time, which is between 0 and 59, so used `k` will also be in this range. With the knowledge of value of `k`, we can calculate private key and sign every message we want. As there aren't many possible values of `k`, we can brute force it by assuming it is equal to 1 and check if further calculations will give flag.

```python
from pwn import *
import json
from ecdsa.ecdsa import Public_key, Private_key, Signature, generator_192
from Crypto.Util.number import bytes_to_long, long_to_bytes
from ecdsa.ecdsa import generator_192

FLAG = "crypto{?????????????????????????}"
g = generator_192
n = g.order()

def sha1(data):
    sha1_hash = hashlib.sha1()
    sha1_hash.update(data)
    return sha1_hash.digest()

def sign_unlock(priv_key):
    pubkey_obj = Public_key(g, g * priv_key)
    privkey_obj = Private_key(pubkey_obj, priv_key)
    msg = "unlock"
    hsh = sha1(msg.encode())
    sig = privkey_obj.sign(bytes_to_long(hsh), 420)
    return {"msg": msg, "r": hex(sig.r), "s": hex(sig.s)}

conn = remote('socket.cryptohack.org', 13381)
conn.recvline()

# Attack:
itr = 0
while(1):
    print(itr)
    itr += 1
    conn.sendline(b'{"option": "sign_time"}')
    rec = conn.recvline().decode()
    sign = json.loads(rec)
    ind = sign['msg'].find(':')
    k = 1
    r = int(sign['r'], 16)
    s = int(sign['s'], 16)
    msg = bytes_to_long(sha1(sign['msg'].encode()))

    priv_key = (((s * k - msg) % n) * pow(r, -1, n)) % n
    payload = sign_unlock(priv_key)
    payload['option'] = 'verify'
    payload = json.dumps(payload)

    conn.sendline(payload.encode())
    print(conn.recvline())
```

Flag: `crypto{ECDSA_700_345y_70_5cr3wup}`

---

## Moving Problems

> I've learnt that when life gives you lemons, if you look at things the right way they taste just like pairs.

Given curve is vulnerable to [MOV attack](https://risencrypto.github.io/WeilMOV/). The general idea is to reduce initial elliptic curve discrete logarithm problem to easier finite field discrete logarithm. Math behind it is too complicated for me and I don't get it, so I used [implementation from internet](https://ctftime.org/writeup/33964). I used sage for finite field DLP, but it needs some time to calculate.

```python
# sage
import hashlib
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

def decrypt_flag(shared_secret):
    iv = bytes.fromhex('eac58c26203c04f68d63dc2c58d79aca')
    ct = bytes.fromhex('bb9ecbd3662d0671fd222ccb07e27b5500f304e3621a6f8e9c815bc8e4e6ee6ebc718ce9ca115cb4e41acb90dbcabb0d')

    sha1 = hashlib.sha1()
    sha1.update(str(shared_secret).encode('ascii'))
    key = sha1.digest()[:16]
    cipher = AES.new(key, AES.MODE_CBC, iv)

    pt = unpad(cipher.decrypt(ct), 16)
    return pt

p = 1331169830894825846283645180581
a = -35
b = 98

E = EllipticCurve(GF(p), [a, b])
G = E((479691812266187139164535778017, 568535594075310466177352868412))
Alice = E((1110072782478160369250829345256, 800079550745409318906383650948))
Bob = E((1290982289093010194550717223760, 762857612860564354370535420319))

# https://ctftime.org/writeup/33964
order = G.order()
k = 1
while (p**k - 1) % order:
    k += 1
# k = 2

Ee = EllipticCurve(GF(p^k, 'y'), [a, b])
Ge = Ee(G)
Ae = Ee(Alice)

T = Ee.random_point()
M = T.order()
d = gcd(M, G.order())
Q = (M//d)*T

assert G.order() / Q.order() in ZZ

N = G.order()

a = Ge.weil_pairing(Q, N)
b = Ae.weil_pairing(Q, N)

# After a few minutes: na=29618469991922269
na = b.log(a)
assert na*G == Alice

priv_key = (na*Bob).xy()[0]

print(decrypt_flag(priv_key))
```

Flag: `crypto{MOV_attack_on_non_supersingular_curves}`

## Thank you for reading! :)

Check out my [GitHub](https://github.com/JakubZojdzik) and other articles