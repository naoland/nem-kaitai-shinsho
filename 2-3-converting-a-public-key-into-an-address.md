# 2.3 パブリックキーからNEMアドレスを作る

前回飛ばしてしまったNEMアドレス生成の部分です。この部分は、ぱっと見簡単そうですが、ものすごく難解な事が書かれています。実行例の検証のために、自分でコードを書く（ほぼ先人のコピペ）ところまでなんとかたどりつきました。自分のパブリックキーからNEMアドレス変換して確認済み。

----

# 2.3 公開鍵からNEMアドレスを作る方法

![](https://s3-ap-northeast-1.amazonaws.com/nem-social/blog/0/7000/7600/7666/1542918235%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202018-11-23%205.15.06.png)

パブリックキーをNEMアドレスに変換する方法は以下の通りです；

1. パブリックキーに256ビットのSha3を実行してハッシュを得る（この時点でパブリックキーは暗号化される）
2. 1で得られた256ビットのハッシュに、更に160ビットのRipemdを実行してハッシュを得る（ビットコインと同様の2重ハッシュ）
3. Ripemdで得られた160ビットのハッシュの先頭にバージョンを表す1バイト（0x68または0x98）をくっつける（21バイトになる）
4. もう一度256ビットのSha3を用いてハッシュ化し、最初の4バイトを検証用のチェックサムとして使う（残りのハッシュは使わない）
5. 3の21バイトと4の4バイトをくっつけると25バイトになる
6. それをbase32でエンコードしてNEMアドレスとする

![](https://s3-ap-northeast-1.amazonaws.com/nem-social/blog/0/7000/7600/7666/1542925479%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202018-11-23%207.23.36.png)

ここから実行例が載せられているのですけれども、最初のところでつまづきました。まず、X, Y, Zの3つのパブリックキーからひとつのパブリックキーを圧縮によって得る方法が書かれていません。パブリックキーの生成には楕円関数が使われるので、とりあえずその上での足し算とでもしておきます（分かる人はコメントいただけるとうれしいです）。以下では、compressedキーからスタートします。

例：

まず実際にsha3_256を実行してみましょう。このサイトを利用させてもらいます。

パブリックキー：c5247738c3a510fb6c11413331d8a47764f6e78ffcdb02b6878d5dd3b77f38ed

得られたハッシュ：278ac44bc6051c979284970c3cc0cbca75666f2fe34b0d70ec77d4c5d3d04529

あれ？Technical Referenceのexampleと合わない、、


![](https://s3-ap-northeast-1.amazonaws.com/nem-social/blog/0/7000/7600/7666/1542922723%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202018-11-23%206.28.16.png)

ラズパイで確認しても、やっぱり合わない。

このあたりを読んでみると、NEMが使っているsha3は、特殊なやつみたいなのです。あとビッグエンディアン、リトルエンディアンの問題もありそうです。はまると時間を浪費するだけなので、とりあえずNISAPIを叩いてみることにします。

http://alice2.nem.ninja:7890/account/get/from-public-key?publicKey=c5247738c3a510fb6c11413331d8a47764f6e78ffcdb02b6878d5dd3b77f38ed

結果：

![](https://s3-ap-northeast-1.amazonaws.com/nem-social/blog/0/7000/7600/7666/1542924711%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202018-11-23%207.08.03.png)

無事NEMアドレスを取得できました。

つまりどういうことかと言うと、NEMのNISAPIは、特殊なSha3 (Keccak)を使っているということです。調べてみると、KeccakというNEMが採用しているSha3は、設計者によるオリジナルと言えるもので、一般に公開されたSha3はKeccakをわずかに改造したもののようです。NEMは「本物」にこだわったという事ですね。


検証のために上のplanethoukiさんのコードを参考（ほぼコピペ）にして、公開鍵からNEMアドレスを導出するpythonコードを書いてみました。

```python
#From Public Key to NEM Address

import hashlib
import sha3
import struct
import binascii
import base64

pubKey = 0xc5247738c3a510fb6c11413331d8a47764f6e78ffcdb02b6878d5dd3b77f38ed
pubKey_bin = binascii.unhexlify(format(pubKey,'064x'))
print('#2 pubKey: %064x' % pubKey)

h_bin = sha3.keccak_256(pubKey_bin).digest()
sha3PubKey = binascii.hexlify(h_bin)
print('#3 sha3 pubKey: %s' % sha3PubKey)

ripemd160PubKey = hashlib.new('ripemd160')
ripemd160PubKey.update(h_bin)
print('#4 ripemd: %s' % ripemd160PubKey.hexdigest())

versionPubKey = '68' + ripemd160PubKey.hexdigest()
print('#5 version: %s' % versionPubKey)

sha3check = sha3.keccak_256(binascii.unhexlify(versionPubKey)).hexdigest()
print('#6 checksum: %s' % sha3check)
print('#7 4-byte checksum: %s' % sha3check[0:8])

address_raw = versionPubKey + sha3check[0:8]
print('#8 raw address: %s' % address_raw)

print('#9 address: %s' % base64.b32encode(binascii.unhexlify(address_raw)))
```

実行結果

![](https://s3-ap-northeast-1.amazonaws.com/nem-social/blog/0/7000/7600/7666/1542930530%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202018-11-23%208.48.22.png)

自分のパブリックキーを使って自分のNEMアドレスが作れるかどうかチェック

![](https://s3-ap-northeast-1.amazonaws.com/nem-social/blog/0/7000/7600/7666/1542930878%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202018-11-23%208.52.15.png)


**合っているようです！**

このコードがあれば、公開鍵を使っていつでもNEMアドレスを作ることができますね。使いみちはあまりないですけど、、、。

----

このような感じで、Technical Referenceは専門書に近く、読むだけ、訳すだけでは、本質的なところはまるで分からないということになりそうです。実際に確かめて、自分でコーディングして、ようやくちょっと分かる程度。うーん、先が思いやられます。


よくわからない文章を、最後まで読んでくださって本当にありがとうございます。またアドレス変換コードを公開してくださったplanethoukiさんに、深く感謝いたします。