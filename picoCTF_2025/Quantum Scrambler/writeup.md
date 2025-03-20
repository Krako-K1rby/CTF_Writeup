# Quantum Scrambler (rev) 200pt

## Question
We invented a new cypher that uses "quantum entanglement" to encode the flag. Do you have what it takes to decode it?

## Solution
渡されたソースコードを見ると、フラグの各文字を16進数文字にした後にscrambleという関数でコネコネして、その結果を出力している。

サーバに接続して出力された結果を見てみると、ネストされたリストになっていることが分かる。

ソースコードと結果を見てみると、2つ目にネストしているリストの最初と最後の要素がフラグの各文字であることが分かる（最後のリストは要素1つだが、それもフラグの文字）。

なので、2つ目にネストしているリストの最初と最後の要素を抜き出すプログラムを書いて実行すると、フラグを取ることができた。

```python
from pwn import *
import ast

io = remote('verbal-sleep.picoctf.net', 63518)

ret = io.recvall()

io.close()

cypher = ast.literal_eval(ret.decode().strip())

result = ['0x70', '0x69']

for i in range(1, len(cypher)):
    if len(cypher[i]) == 3:
        result.append(cypher[i][0])
        result.append(cypher[i][2])
    elif len(cypher[i]) <= 2:
        result.append(cypher[i][0])

ans = ""

for x in result:
    ans += chr(int(x, 16))

print(ans)
```

## Flag
picoCTF{python_is_weirdef8ea0cf}