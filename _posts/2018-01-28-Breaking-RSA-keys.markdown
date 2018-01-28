---
layout: post
title:  "Breaking RSA keys"
date:   2018-01-28 11:18:29 +0200
categories: blog
---
# Breacking RSA keys
RSA is the most famous asymmetric cipher. He was created in 1977 by three cryptographic researcher (Rivest, Shamir y Adleman).
# How works RSA?
How you know, the asymmetric cipher use one pair of keys for cipher and decrypt messages. Now i go to explain how work all this process.
In first place is necessary select two prime numbers that we know as number "p" and number "q".

```
p=7
q=11```

The next step is calculate N that is the result of p*q:

 ```
 N=7*11=77```

 Now we need calculate phi(N), 'phi' is the Euler's phi function. This function counts the positive integers up to a given integer n that are relatively prime to n.

 ```
 phi(N)=(p-1)*(q-1)
 phi(N)=(6*10)=60
 ```
 In forth place we need select a public exponent, more later we know why this name, this number is very simple of select. It is a aleatory number between 3 and phi(N) that the same time will be relative prime with phi(N).

 In this case we select the number 13

 ```
 mcd(13,60)=1
 e=13
 ```
 The next step is found a inverse multiplicative of 13 in module phi(N),the number 'd', so 'mod 60 (13*d)=1':

 ```
 (13*37)%60=1
 d=37
 ```
 The number 'd' is a private exponent. Now we have all necessary numbers for compose the private and public keys.
 The private key is the number N and exponent 'd', and the public key is the number N and exponent 'e'.
 PrivateK(77,37)
 PublicK(77,13)

 # How we can cipher and decrypt messages?

 These process is very easy. Cipher a message, for example the number 2, we need calc 2^(public exponent) in modulus N, in this case 2^13 in mod N:

 ```
 CM(M)=mod N (M^e)
 CM(2)=(2*13)%77
 CM(2)=30
 ```

 For decrypt a message is the same process, we need the cipher message, in this case '30' and calculate 30^(private exponent) in modulus N:

 ```
 DM(CM)=mod N (CM^d)
 DM(30)=(30^37)%77
 DM(30)=2
 ```

# Is time to play :)

Now I explain how you can generate a private key with base in one short public key for example one public key with 128bits.

The next step is check if the modulus of the key is a short number. (The modulus is the number N that is p*q, if you can factorize N you have the two prime factors, and you can calculate all parameters)

![Openssl command](https://github.com/dsfau/dsfau.github.com/blob/master/_posts/_img/openssl_command.png?raw=true)

```
openssl rsa -pubin --modulus --text < rsa128.pub
```

 Now we need calculate the decimal number of the modulus, because it is writed in hexadecimal, for make the conversion you can use python.

![Python hex to decimal](https://github.com/dsfau/dsfau.github.com/blob/master/_posts/_img/hex_to_dec.png?raw=true)

 ```
 python -c "print 0xXXXXXXXXXXXXXX"
 ```

Now we need factorize this number, we can use primefac for example that is a library for python, and factorize the number in own PC but previously we can consult if the number is already factorized in factordb.com

![Factordb](https://github.com/dsfau/dsfau.github.com/blob/master/_posts/_img/factordb.png?raw=true)

In this case the number was factorized, now we need calculate the number phi(N) and the number 'd' (private exponent).
For calculate phi(N) is very easy if you have the two factors 'p' and 'q':

![Phi of N](https://github.com/dsfau/dsfau.github.com/blob/master/_posts/_img/phi_of_n.png?raw=true)

```
phi(N)=(p-1)*(q-1)
```

Now we need calculate the inverse multiplicative and for this task I use this code in python:

```python
	def egcd(a, b):
	    if a == 0:
	        return (b, 0, 1)
	    g, y, x = egcd(b%a,a)
	    return (g, x - (b//a) * y, y)

	def modinv(a, m):
	    g, x, y = egcd(a, m)
	    if g != 1:
	        raise Exception('No modular inverse')
	    return x%m
	d=modinv(65537, 255581954141729999155835996851383671760)
	print d
```

Now we have all necessary parameters N, the private exponent 'd' and the public exponent 'e'. Only need reconstruct the private key. We can use the python library Crypto for make these.

``` python
>>> e=65537L
>>> n=255581954141729999188016320977612923849L
>>> d=209993179316667852775955340669060004073L
>>> private_key = RSA.construct((n, e, d))
>>> print private_key.exportKey()
-----BEGIN RSA PRIVATE KEY-----
MGMCAQACEQCeYKyhjGPmnRe+Z6WEepujAgMBAAECEQCd+zcYam8YkJdxEKUqCjjp
AgkAyPDk9NfPaj0CCQDJxibrAyDbXwIJAKCu+ffNB8DlAggx+YoaGyRAMQIId2xi
Jmrj+bs=
-----END RSA PRIVATE KEY-----
```

Now we can read all messages cipher that use this pair of keys.
Regards!
