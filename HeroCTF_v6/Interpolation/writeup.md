# Interpolation (crypto) 83/670 (12.4%)

## Question

Has missing data really ever stopped anyone ?

## Analysis

### chall.sage

```python
#!/usr/bin/sage
import hashlib
import re

with open("flag.txt", "rb") as f:
    FLAG = f.read()
    assert re.match(rb"Hero{[0-9a-zA-Z_]{90}}", FLAG)

F = FiniteField(2**256 - 189)
R = PolynomialRing(F, "x")
H = lambda n: int(hashlib.sha256(n).hexdigest(), 16)
C = lambda x: [H(x[i : i + 4]) for i in range(0, len(FLAG), 4)]

f = R(C(FLAG))

points = []
for _ in range(f.degree()):
    r = F.random_element()
    points.append([r, f(r)])
print(points)

flag = input(">").encode().ljust(len(flag))

g = R(C(flag))

for p in points:
    if g(p[0]) != p[1]:
        print("Wrong flag!")
        break
else:
    print("Congrats!")
```

与えられたコードについて理解していきます。\
まず、flag.txtからフラグを読み出してくるところからフラグの{}の中が90文字であることが分かります。

次に、
``` python
F = FiniteField(2**256 - 189)
R = PolynomialRing(F, "x")
H = lambda n: int(hashlib.sha256(n).hexdigest(), 16)
C = lambda x: [H(x[i : i + 4]) for i in range(0, len(FLAG), 4)]

f = R(C(FLAG))
```

では、係数が有限体 $F_{2^{256}-189}$ の要素である多項式 $R$ を宣言した後に、\
フラグの前から4文字ずつ取り出した文字列をSHA-256でハッシュ化して整数に変換し、
それらを係数とした多項式 $f$ を宣言しています。

そして、
```python
points = []
for _ in range(f.degree()):
    r = F.random_element()
    points.append([r, f(r)])
print(points)
```
$f$ の次数分、多項式にランダムな値を代入して、その結果との組をユーザに与えています。

最後に、
```python
flag = input(">").encode().ljust(len(flag))

g = R(C(flag))

for p in points:
    if g(p[0]) != p[1]:
        print("Wrong flag!")
        break
else:
    print("Congrats!")
```
とあることから、 $f$ と同じ係数になるような入力を求められていることが分かります。

## Solution
この問題は、問題文から分かる通り多項式補間を行うことで解くことができます。\
多項式補間は $n+1$ 個の点が与えられたときに、それらを通る $n$ 次以下の多項式を求めることができるというものです。

今回の場合は、 $n$ 個の点が与えられていますが、 $n$ 次の多項式を求める必要があるため、多項式補間をそのままでは行うことができません。\
しかし、フラグの最初が**Hero**であることが分かっているため、これを利用すると $f$ の最低次数係数が$$であると分かります。\
すると、$f$ に $0$ を代入した結果は最低次数係数の値になるため、点 $(0, )$ がもう一つ増えることになります。\
したがって、$n+1$ 個の点が与えられることになるため、多項式補間を行い $f$ の係数を復元することができます。

これで $f$ の係数は求まりましたが、そこからSHA-256のハッシュ化する前の文字列を求めなくてはいけません。\
幸いにも4文字かつ文字の種類がアルファベットと数字、{}_に制限されているため、全探索でハッシュ化する前の文字列を求めます。

以上の処理をコードに起こし、実行するとフラグが得られました。

### solver.sage

```python
#!/usr/bin/sage
import hashlib
import itertools
import string

with open('points.txt', 'r') as f:
    points = eval(f.read())

F = FiniteField(2**256 - 189)
R = PolynomialRing(F, "x")

points.append([0, int(hashlib.sha256(b'Hero').hexdigest(), 16)])

f = R.lagrange_polynomial(points)

characters = string.ascii_lowercase + string.ascii_uppercase + string.digits + '_{}'

combinations = [''.join(comb) for comb in itertools.product(characters, repeat=4)]

hash_list = []
for comb in combinations:
    hash_list.append([comb, int(hashlib.sha256(comb.encode()).hexdigest(), 16)])

res = ""

for coef in f.coefficients():
    for s,h in hash_list:
        if coef == h:
            print(s)
            res += s

print(res)
```

## Flag
`Hero{th3r3_4r3_tw0_typ35_0f_p30pl3_1n_th15_w0rld_th053_wh0_c4n_3xtr4p0l4t3_fr0m_1nc0mpl3t3_d474}`