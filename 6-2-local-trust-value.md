# 6.2 ノード信頼度リスト

こんにちは。久しぶりのNEM勉強会です。

NISノードは、他のノードと常に通信を繰り返しながら、ブロックチェーンをつないでいます。相手のノードが落ちていたり、不正な値を返してきたりした場合に、そのノードとの通信を避けて、より良いノードと通信しようとします。今回は、そこで使われるノードの信頼度を示す数値の計算の仕方についてです。

----

6.2 ローカル信頼度

それぞれのノード（NISがインストールされた計算機）は、あらかじめ設定された信頼できるノードのセット「P」からスタートします。

これはコンフィグファイルに記載されていて、必要に応じて使用者が改変することができます。

ノードは、この信頼できるノードリストPを読み込み、リストに載っているノードとの通信を通して、他のノードの存在を知ります。

最初のノードのリストを読み込み後、それを元にノードは信頼度値のベクトル（数値の羅列）pを作ります。pの要素、pjの値は下記の式で計算されます。  
![](https://s3-ap-northeast-1.amazonaws.com/nem-social/blog/10000/11000/11600/11636/1548130255%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202019-01-22%2013.10.25.png)

pjはノードjがどのくらいの信頼度を持っているかを示します。

```
最初のリストPにIPアドレスが入っていれば、Pベクトルの絶対値分の１がそのノードに与えられるようです。最初はすべて平等です。リストに入っていないノードの信頼度は０です。

Pの絶対値で割っているためベクトルの長さも１（単位ベクトル）になってます。でも絶対値よりpjの合計値Σのほうがしっくりくるような気もします。

下が、実際にNISが最初の信頼度を計算しているところのログです。ローカルのコンフィグファイルの該当部分にはデフォルトではノード指定がされておらず、どうやらorg.nem.peer.trust.CachedTrustProviderに問い合わせているみたいです。自分でコンフィグリストにIPアドレスのリストを書き加えれば、それが追加あるいは置き換えて用いられるようですが、あまりやる人はいないでしょうね。ログの中では初期リストにある、13ノードについてまず信用度を計算（1/√13 or 1/13?）し、そのあと世界中のノードに問い合わせをして、ACTIVEかどうかを確認しています。
```

```
2018-12-03 00:39:31.550 情報 calculating trust values (org.nem.peer.trust.CachedTrustProvider a)
2018-12-03 00:39:31.562 情報 trust calculation finished (13 values) (org.nem.peer.trust.CachedTrustProvider a)
2018-12-03 00:39:32.526 情報 Updating "Node [Saffron <NDYO7LWSS6NFJLCIIAPVC5IVOE5AFZTSR46KLBBH>] @ [148.251.112.53]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:32.623 情報 Updating "Node [NB5I24Y6VFOCYJRUAIRNFRNGE4OAQVOTWDB266DW <NB5I24Y6VFOCYJRUAIRNFRNGE4OAQVOTWDB266DW>] @ [52.68.132.108]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:32.699 情報 Updating "Node [USA-NM-ABQ <NCEJ7Y2J3YPHBQZTJFQ2A4XCLPT2XOTUJFTBOVQW>] @ [65.132.7.227]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:32.766 情報 Updating "Node [SUPBar.nemspace <NCSOJ5AEDPKHAHFCY5M6JJURMVF6N37PWZXYAGST>] @ [74.208.244.49]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:33.537 情報 Updating "Node [DRAGON <NBBVBVNLFFYFPU7OPT67HPPKQN7U3JEQC7KQXLIJ>] @ [110.44.135.87]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:33.601 情報 Updating "Node [NEMron <NDKPBH57BFB7EG5QF2TKSDJTFOHNGSRQSSAUEQM7>] @ [104.128.68.205]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:33.679 情報 Updating "Node [Andarivadi Nem Node <NCSCXLFZ3I3BIJVM2YOK2URDUJ3INAOPYDPLBKY4>] @ [45.76.174.212]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:33.699 情報 Updating "Node [Bengal Tiger <NAAIYOHLT7GXYDYBXP3FCSEGDTCML5AKP4CXBUPJ>] @ [139.59.13.11]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:33.959 情報 Updating "Node [Pit Man <NDYULYDMGSWQWBNITFMAY2ZZ4HKMG2PVCQ3HHWRD>] @ [51.141.28.95]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:33.972 情報 Updating "Node [[c=#e1a92b]A[/c][c=#41ce7c]f[/c][c=#8e8e8e]r[/c]ica <NAGJ6NW2DQKPVIDD5YACST4ET5PEVWX6TDJTPIN3>] @ [88.198.125.199]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:33.980 情報 Updating "Node [eu.2 <NBTPS3GSFFDRPIC7LR6KVRQRISR7BMQWXQV6RNFA>] @ [213.136.86.202]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:34.006 情報 Updating "Node [xem-malaysia <NCF3GXYEGYOC4D6YY43LGNBA5ZHDQJNU32QTF5E7>] @ [210.16.120.28]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:34.064 情報 Updating "Node [[c=#e1a92b]N[/c][c=#41ce7c]E[/c][c=#8e8e8e]M[/c]pragt3 <NAMXWGYWH53XVFLOXELV7B5ETHGLEMYX6KLGQAUW>] @ [78.47.148.104]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:34.079 情報 Updating "Node [Singa02 <ND64JBEUHPPGISSIDWXYWNMVHOI45QOSQNZ66YZP>] @ [45.32.134.182]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:34.097 情報 Updating "Node [mnbhsgwgamma <NAVCQA4A35FXDNKSLN6NIODJPIUCQISUWIEN5Q7S>] @ [150.95.136.134]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:34.179 情報 Updating "Node [liang2 <NBTV56HVTNIXBW3HEE6ETFN4ELOTVCFQMW23IEZN>] @ [160.16.63.95]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:34.275 情報 Updating "Node [chi <NDUNAVOFOYYQYSWUDVF445GKV5OXAGN24CGMMZEK>] @ [153.122.13.95]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:34.721 情報 Updating "Node [NAIMGGJFVTZBOCRMUBIA53EJLAM2ZKDSWQZ2DPFT <NAIMGGJFVTZBOCRMUBIA53EJLAM2ZKDSWQZ2DPFT>] @ [sky.loxal.net]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:37.592 情報 Updating "Node [TIME <NCNDBFCX3KQ4PM6SNK2NLE6CWW5PZFNBE3YD4ZQI>] @ [149.56.47.0]" -> ACTIVE (org.nem.peer.services.NodeRefresher update)
2018-12-03 00:39:43.600 情報 INDIRECT warning (BUSY) encountered while communicating with <Node[Hi,IamAlice4<NDNQWXFKVVLA6RVUIGLG6AZVF3FRBV7L5WMIZ3ZT>] @ [209.126.98.204]>: java.util.concurrent.CompletionException: org.nem.core.connect.BusyPeerException: java.net.SocketTimeoutException (org.nem.peer.services.NodeRefresher a)
```

ノードiがある程度ネットワークとコミュニケーションを取ったあとで、ノードリストをアップデートします。

その時に改めてローカル信頼度が計算されます。  
![](https://s3-ap-northeast-1.amazonaws.com/nem-social/blog/10000/11000/11600/11636/1548138486%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202019-01-22%2013.10.34.png)

```
success(i, j)は、ノードiからノードjへの情報通信が成功した場合。failed(i, j)は逆に失敗した場合の累積回数です。どちらかの結果が返ってきたら、success(i, j)やfailed(i, j)が加算されているはずなので、sijはノードiからjに向けての通算のこれまでの通信成功率になります。まったく通信しなかった場合は、初期値を引き継ぎます。

最初の問い合わせでsuccessならsij = 1, failedならsij = 0, それ以外ならpjのままです。
```

そして、sijのベクトルをノーマライズして（`ここではなぜかベクトルの絶対値ではなく、合計値で割っている`）合計数１のベクトルに変換します。

![](https://s3-ap-northeast-1.amazonaws.com/nem-social/blog/10000/11000/11600/11636/1548138706%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202019-01-22%2013.10.40.png)

得られたcijをまとめたベクトルciがノードiの持つローカル信頼度ベクトルです。

----

```
コメント：

NISノードは自分が持つノードリストを読み込んだあと、他のノードと同期をしながら、信頼度データをアップデートしていくようです。

初期に使われるノードは、NISのプログラムにプレインストールされていて、コンフィグファイルでの変更も可能になっています。

ログに示した例では、初めてNISを立ち上げた時に、１３個のノードから最初のローカル信頼度が計算されています。

そして、すぐに19個のノードとの通信を試みて、ノードが生きている（ACTIVE）ことを確認しているように見えます。

その後のローカル信頼度計算にはこれらのノードが使われるようですが、なぜ１３個より増えたかは謎です。

 

NEMはすべてのノードが同じ信頼度データリストを共有するのではなく、それぞれが個別にノードリストを持つ点に特徴があります。

このノードリストをpeer to peerで交換しながら、ネットワーク全体を網羅していくようになっています。

 

要するに「それぞれのノードが自分で確かめてる！」ってことですね。
```

次回は、それぞれのノードが持つローカル信頼度を、peer to peerでお互いにやりとりする仕組みのお話しになります。

いつも読んでくださってありがとうございます！
