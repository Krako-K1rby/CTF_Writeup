# Basic_Crypto1

# Question
This Problem Basic_Crpyto(Roman emperor's cipher)

FLAG FORMAT(A~Z) and empty is "_"

DH{decode_Text}

# Solution
問題文からシーザー暗号であることが分かるため、[シーザ暗号の解読を行うサイト](https://cryptii.com/pipes/caesar-cipher)を使います。

その結果、アルファベットを右に23文字ずらすとBASIC CRYPTO DREAMHACKという文字列が出てきたので、これをDH{}で囲むとフラグになりました。

# Flag
DH{BASIC_CRYPTO_DREAMHACK}