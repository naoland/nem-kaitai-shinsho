# 番外編: 初ハーベスト

まずは、ブロックチェーンの中身をお見せします。

```
{"timeStamp":120430506,
"signature":"d84844cf4cd19f9e0a85c43e94939acf615f81893570617bf15df923d72bb8c158e532446f4a7e7a484a2cd03d2668a69ae130e3b8ae177b6b12353380d66805",
"prevBlockHash":{"data":"912c11ff34205a82871acdd81d5f8a155251c264eb36ec39f7561fd39c16637f"},
"type":1,
"transactions":[
	{"timeStamp":120430442,
		"amount":50000,
		"signature":"94286f7c185f66d9f87d150b78d1398dbb0d308128f332dbfd8b2c6b341a54ce4652a1d67608ae297f9d6204ad6f96528ecd9fd6d2d5b8a55fd512ae0e432a0c",
		"fee":50000,←手数料0.05XEM
		"recipient":"NDBZRFWARBAQSX5GDBRU2QJ7QTCPVRI5WV6RY74A",
		"type":257,
		"deadline":120516842,
		"message":{},
		"version":1744830465,
		"signer":"d22b047b670fd32ad9fa421a37415ab0677a786b916cad34820bcd395066cd49"},
	{"timeStamp":120430456,
		"amount":50000,
		"signature":"f129388abf27e8476004f01f9defb901b8f68923f540bf96fdfaccdddd62a0063c3ee5e673c3d0a684d4e943b52c8424fbc6621111e7436302eded0872ca4e05",
		"fee":50000,←手数料0.05XEM
		"recipient":"NB4GJ3ZOLUBMQSTDTQKQUQQJ6XR4XMRAEZJLRUY2",
		"type":257,
		"deadline":120516856,
		"message":{},
		"version":1744830465,
		"signer":"d22b047b670fd32ad9fa421a37415ab0677a786b916cad34820bcd395066cd49"},
	{"timeStamp":120430461,
		"amount":1000000,
		"signature":"d8f4f3e655290045c015051edf704fa5365006e47127bc5831bda4f3ae6ed18063af9d54d8fdddfefedfb1bb50b69e4107cb4859e84380ae730f8c881a292206",
		"fee":50000,←手数料0.05XEM
		"recipient":"NB5HJP6S2BWEMASR54YKVJJ2FVHCGPHNWRP5FZRA",
		"mosaics":[{"quantity":2002250,
		"mosaicId":{"namespaceId":"native",
		"name":"coin"}}],←native coin というモザイクが送られた
		"type":257,
		"deadline":120516861,
		"message":{},
		"version":1744830466,
		"signer":"d22b047b670fd32ad9fa421a37415ab0677a786b916cad34820bcd395066cd49"},
	{"timeStamp":120430461,
		"amount":1000000,
		"signature":"3469f01cda1b8b3e603e9d138ffa672cd60ded50e3f94a7f7f97a9334de6f2bf0f72912e5558226b0a566673fca529de54ba8e36aa574f8aab468e4dc36e6403",
		"fee":50000,←手数料0.05XEM
		"recipient":"NAX73DHUTZNKKNAVYLPWB6735YDHRFFR7HNPIP5F",
		"mosaics":[{"quantity":5005250,
		"mosaicId":{"namespaceId":"native",
		"name":"coin"}}],←native coin というモザイクが送られたようです
		"type":257,
		"deadline":120516861,
		"message":{},
		"version":1744830466,
		"signer":"d22b047b670fd32ad9fa421a37415ab0677a786b916cad34820bcd395066cd49"}
	],
"version":1744830465,
"signer":"5fbbf7e396d90ac2b3d3ff91d6f84561337428a69dc825edc6027a7dc46c8843",←僕のデリゲートキー
"height":1989616}
```

ついにやりました！初ハーベストです。

昨年からNISを立ち上げつつ、こんなローカルサーバーでもハーベストできるのかな〜と待っていたら、ようやく0.2XEMハーベストできました。

ブロック：1989616

トランザクション数：4

報酬：0.2XEM (0.05 x 4)

singerのところに、自分の委任デリゲートキーがちゃんと入っています。

 

NISの該当部分のログを見たところ、そっけないもんでした。おめでとう！とかyay!とか出るかと思ってたんですが、なんもないです。

```
2019-01-21 06:01:12.302 情報 received 1 blocks (4 transactions) in 17 ms from remote (4250 μs/tx) (org.nem.nis.sync.BlockChainUpdater c)
ここでハーベストされた？→2019-01-21 06:01:12.318 情報 1 harvesters are attempting to harvest a new block. (org.nem.nis.ld.gah cH)
2019-01-21 06:01:12.422 情報 validated 1 blocks (4 transactions) in 117 ms (29250 μs/tx) (org.nem.nis.sync.BlockChainUpdateContext fz)
2019-01-21 06:01:12.422 情報 new block's score: 108209381871612 (org.nem.nis.sync.BlockChainUpdateContext a)
2019-01-21 06:01:12.454 情報 chain update of 1 blocks (4 transactions) needed 32 ms (8000 μs/tx) (org.nem.nis.sync.BlockChainUpdateContext fz)
```

----

とりあえず自分で立ち上げたNISでもちゃんとハーベストできるという確認ができてよかった。

