---
layout: post
title:  "Attacks for stream ciphers"
date:   2017-04-3 11:18:29 +0200
categories: blog
---
# Attacks for stream ciphers (XOR)

The difference between stream cipher and block ciphers is that the stream cipher is used bit by bit in a string while a block cipher is used in data blocks. One form of stream cipher is the binary operation XOR.
## How it works XOR?
This is the truth table:

| A | XOR | B | OUTPUT |
|:---:|:-----:|:---:|:--------:|
| 0 |  ⊕  | 0 | 0      |
| 0 |  ⊕  | 1 | 1      |
| 1 |  ⊕  | 0 | 1      |
| 1 |  ⊕  | 1 | 0      |

A ⊕ 0 = A

A ⊕ A = 0

(A ⊕ B) ⊕ C = A ⊕ (B ⊕ C)

This operation is sometime called modulus 2 addition. Applying this logic, a string text can be encryped by applying the bitwise XOR operator to every character using a given key.

Encrypt: C(M)=M ⊕ K

 Decrypt: M=C(M) ⊕ K

This cipher is vulnerable to attack if the same key is used twice or more time.
Say we send messages with the same length and both be encrypted with the same key, K, the messages produced are these:

C(A) = A ⊕ K

C(B) = B ⊕ K

If a adversary has intercepted the C(A) and C(B), he can easily calculate C(A) ⊕ C(B).

Thus, if we apply the the commutative property of xor that X ⊕ X = 0, we obtain:

C(A) ⊕ C(B) => (A ⊕ K) ⊕ (B ⊕ K) => A ⊕ B ⊕ K ⊕ K => A ⊕ B

In other words, if anyone intercepts two messages encrypted with the same key, they can recover A xor B, which is a form of running key cipher.
With an average personal computer, such ciphers can usually be broken in a matter of minutes. If one message is known, the solution is trivial.
## Example



```python
li=['a','b','c','d','e','f','g','h','i','j','k','l','m','n','o','p','q','r','s','t','u','w','x','y','z']
dic={}
get_bin = lambda num: format(num, 'b').zfill(5)
for i in li:
    pos=li.index(i)
    dic[i]=get_bin(pos)
print dic
```

    {'a': '00000', 'c': '00010', 'b': '00001', 'e': '00100', 'd': '00011', 'g': '00110', 'f': '00101', 'i': '01000', 'h': '00111', 'k': '01010', 'j': '01001', 'm': '01100', 'l': '01011', 'o': '01110', 'n': '01101', 'q': '10000', 'p': '01111', 's': '10010', 'r': '10001', 'u': '10100', 't': '10011', 'w': '10101', 'y': '10111', 'x': '10110', 'z': '11000'}



```python
A="t e s t o n e"
B="t e s t t w o"
K="p a s s w o r"
A=map(lambda l: dic[l],A.split())
B=map(lambda l: dic[l],B.split())
K=map(lambda l: dic[l],K.split())
print A
print B
print K
```

    ['10011', '00100', '10010', '10011', '01110', '01101', '00100']
    ['10011', '00100', '10010', '10011', '10011', '10101', '01110']
    ['01111', '00000', '10010', '10010', '10101', '01110', '10001']



```python
A=0b10011001001001010011011100110100100
B=0b10011001001001010011100111010101110
K=0b01111000001001010010101010111010001
```

Now we go to encrypt the message A


```python
EA=A ^ K
print "A --> Dec: {0}, Bin: {1}".format(A, bin(A))
print "EA --> Dec: {0}, Bin: {1}".format(EA, bin(EA))
```

    A --> Dec: 20554824100, Bin: 0b10011001001001010011011100110100100
    EA --> Dec: 30199049333, Bin: 0b11100001000000000001110110001110101


Now decrypt the message:


```python
DA = EA ^ K
print "A --> Dec: {0}, Bin: {1}".format(A, bin(A))
print "DA --> Dec: {0}, Bin: {1}".format(DA, bin(DA))
```

    A --> Dec: 20554824100, Bin: 0b10011001001001010011011100110100100
    DA --> Dec: 20554824100, Bin: 0b10011001001001010011011100110100100


But what happen when two messages be cipher with the same key?


```python
EA = A ^ K
EB = B ^ K
print "EA --> Dec: {0}, Bin: {1}".format(EA, bin(EA))
print "EB --> Dec: {0}, Bin: {1}".format(EB, bin(EB))
```

    EA --> Dec: 30199049333, Bin: 0b11100001000000000001110110001110101
    EB --> Dec: 30199028607, Bin: 0b11100001000000000001001101101111111



```python
EAxorEB = EA ^ EB
AxorB = A ^ B
print "EAxorEB --> Dec: {0}, Bin: {1}".format(EAxorEB, bin(EAxorEB))
print "AxorB   --> Dec: {0}, Bin: {1}".format(AxorB, bin(AxorB))
```

    EAxorEB --> Dec: 30474, Bin: 0b111011100001010
    AxorB   --> Dec: 30474, Bin: 0b111011100001010

The result of calculate XOR of EA and EB or A and B is the same, this property permit to a attacking obtain the plain text with a very little calcs.

If one of the messages is known by the adversary this process is very easy.

Regards!

