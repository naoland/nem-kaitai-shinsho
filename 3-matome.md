僕は技術系といっても生物系が専門です。にもかかわらず、NEMの技術資料を読んでみようと思いました。その動機は、NEMの技術を利用して、細胞の流通管理をおこなう「仕組み」を作りたいからです。確かにTechnical Referenceの日本語訳はたくさん見つかりました。でも僕には残念ながら理解できませんでした。


僕が預かるものは細胞を提供してくれるドナーの個人情報と遺伝暗号です。お金では買えない価値があります。とりあえず日本語訳を読んだくらいで分かったような気になっていると、あとで大変な事になりそうです。なので、専門家にシステム設計を依頼する前に、NEMについてきちんと理解しないとだめだと思いました。


今回は、1章から３章のまとめをしたいと思います。

----

# 第１〜３章　まとめ　秘密鍵 X 公開鍵 X NEMアドレス

1. 秘密鍵ー公開鍵の組み合わせは1対1だが、NEMアドレスについては非常に低い確率でぶつかる可能性がある。
2. NEMの鍵の数の上限は100000000000000000000000000000000000000000000000000000000000000000000000000000個くらいあるので、枯渇する心配はない。
3. ラズパイのような非力なマシンでも、秘密鍵から公開鍵やNEMアドレスを作ることができる。

わけのわからんブログ記事を4つも使って、わかった結論は、、、、

**「NEMの暗号化プロセスは充分に信頼できるものである。」**

はい、この程度です。でも、今までの僕のあいまいな知識に比べれば、随分と進歩したと思います。理解した事をまとめてpythonコードにしておきます。ラズパイでの実行結果はこんな感じです。検証用のアドレスはplanethoukiさんのところで使われていたものと同じです。

![](https://s3-ap-northeast-1.amazonaws.com/nem-social/blog/0/7000/7900/7913/1543289309%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202018-11-27%2012.27.23.png)

----

過去記事へのリンク  
1 イントロダクション  
2 アカウントとアドレス  
2.3 パブリックキーからNEMアドレスを作る（コードサンプル付き）  
3 暗号化  

上記は不要だと思う（筒井）

今後、ブロックチェーンやProof of Importanceについても、時間を見つけて解析したいと思っています。難しい専門文書の解読にお付き合いくださってありがとうございます！

（おまけ）

これまでの事をまとめたpythonコードです。秘密鍵からNEMアドレスを作ることができます。

```python
#From Private Key to NEM Address
# -*- coding: utf-8 -*-
# sha3はpysha3をインストールしてください（sudo pip install pysha3）
# 参考：https://www.planethouki.me/entry/get_nem_address_from_private_key/
# ed25519コード:https://ed25519.cr.yp.to/python/ed25519.py

import hashlib
import sha3
import struct
import binascii
import base64

#秘密鍵
privKey = 0xaf3c669fc25b6431a3024dc38a8b7338856441f5ac27c1aa0a857d9f893e91a
privKey_bin = binascii.unhexlify(format(privKey,'064x'))
print('秘密鍵: %064x' % privKey)

#基本的な定数
b = 256
q = 2**255 - 19
l = 2**252 + 27742317777372353535851937790883648493

#ed25519用の関数定義と必要な数値の計算
def H(m):
  return hashlib.sha512(m).digest()

def expmod(b,e,m):
  if e == 0: return 1
  t = expmod(b,e/2,m)**2 % m
  if e & 1: t = (t*b) % m
  return t

def inv(x):
  return expmod(x,q-2,q)

d = -121665 * inv(121666)
I = expmod(2,(q-1)/4,q)

def xrecover(y):
  xx = (y*y-1) * inv(d*y*y+1)
  x = expmod(xx,(q+3)/8,q)
  if (x*x - xx) % q != 0: x = (x*I) % q
  if x % 2 != 0: x = q-x
  return x

By = 4 * inv(5)
Bx = xrecover(By)
B = [Bx % q,By % q]

def edwards(P,Q):
  x1 = P[0]
  y1 = P[1]
  x2 = Q[0]
  y2 = Q[1]
  x3 = (x1*y2+x2*y1) * inv(1+d*x1*x2*y1*y2)
  y3 = (y1*y2+x1*x2) * inv(1-d*x1*x2*y1*y2)
  return [x3 % q,y3 % q]

def scalarmult(P,e):
  if e == 0: return [0,1]
  Q = scalarmult(P,e/2)
  Q = edwards(Q,Q)
  if e & 1: Q = edwards(Q,P)
  return Q

def encodeint(y):
  bits = [(y >> i) & 1 for i in range(b)]
  return ''.join([chr(sum([bits[i * 8 + j] << j for j in range(8)])) for i in range(b/8)])

def encodepoint(P):
  x = P[0]
  y = P[1]
  bits = [(y >> i) & 1 for i in range(b - 1)] + [x & 1]
  return ''.join([chr(sum([bits[i * 8 + j] << j for j in range(8)])) for i in range(b/8)])

def bit(h,i):
  return (ord(h[i/8]) >> (i%8)) & 1

def publickey(sk):
  h = H(sk)
  a = 2**(b-2) + sum(2**i * bit(h,i) for i in range(3,b-2))
  A = scalarmult(B,a)
  return encodepoint(A)

def Hint(m):
  h = H(m)
  return sum(2**i * bit(h,i) for i in range(2*b))

def signature(m,sk,pk):
  h = H(sk)
  a = 2**(b-2) + sum(2**i * bit(h,i) for i in range(3,b-2))
  r = Hint(''.join([h[i] for i in range(b/8,b/4)]) + m)
  R = scalarmult(B,r)
  S = (r + Hint(encodepoint(R) + pk + m) * a) % l
  return encodepoint(R) + encodeint(S)

def isoncurve(P):
  x = P[0]
  y = P[1]
  return (-x*x + y*y - 1 - d*x*x*y*y) % q == 0

def decodeint(s):
  return sum(2**i * bit(s,i) for i in range(0,b))

def decodepoint(s):
  y = sum(2**i * bit(s,i) for i in range(0,b-1))
  x = xrecover(y)
  if x & 1 != bit(s,b-1): x = q-x
  P = [x,y]
  if not isoncurve(P): raise Exception("decoding point that is not on curve")
  return P

def checkvalid(s,m,pk):
  if len(s) != b/4: raise Exception("signature length is wrong")
  if len(pk) != b/8: raise Exception("public-key length is wrong")
  R = decodepoint(s[0:b/8])
  A = decodepoint(pk)
  S = decodeint(s[b/8:b/4])
  h = Hint(encodepoint(R) + pk + m)
  if scalarmult(B,S) != edwards(R,scalarmult(A,h)):
    raise Exception("signature does not pass verification")

#--- Main ---

h_bin = sha3.keccak_512(privKey_bin[::-1]).digest()
h_binstrlit = bin(int(binascii.hexlify(h_bin[::-1]),16))

a_right = 0
for i in range(3,253+1):
    a_right += 2**i * int(h_binstrlit[-i-1])

a = 2**254 + a_right

pubKey = scalarmult(B,a)
#print('pubKey Ax: %064x' % pubKey[0])
#print('pubKey Ay: %064x' % pubKey[1])

compressedPubKey_hexstr = binascii.hexlify(encodepoint(pubKey))
print('公開鍵: %s' % compressedPubKey_hexstr)

sha3PubKey = sha3.keccak_256(binascii.unhexlify(compressedPubKey_hexstr))

ripemd160PubKey = hashlib.new('ripemd160')
ripemd160PubKey.update(sha3PubKey.digest())

versionPubKey = '68' + ripemd160PubKey.hexdigest()

sha3check = sha3.keccak_256(binascii.unhexlify(versionPubKey)).hexdigest()

address_raw = versionPubKey + sha3check[0:8]

print('NEMアドレス: %s' % base64.b32encode(binascii.unhexlify(address_raw)))
```
