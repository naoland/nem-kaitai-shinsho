あれ？NEM Technical Referenceの解説はもう終わったんじゃなかったの？と思われる方も多いかと思いますが、実は重要な単元をひとつ飛ばしていました。美味しいものはあとに取っておく性格なのです（嘘）。

 

それは、「4.3 マルチシグトランザクション」です。当時の僕の印象としては、秘密鍵の数が増える分、セキュリティーとリスクが同時に増えるみたいな微妙な機能のため、多くの人がまだその恩恵を受けるに至っていないと考えて、そこだけ飛ばしました。しかし、カタパルトで、アグリゲートトランザクションやアトミックスワップなどの、強力な新機能が実装されてきている事から、その基本になっているマルチシグトランザクションについても勉強しなければという気になりました（遅）。

 

ですので、今回からNEM技術勉強会特別編として、マルチシグに関する項目を説明していきます。

----


# 4.3 マルチシグ関連トランザクション

NEMは、最初からm-of-n（n人のうちm人が賛成したら成立する）タイプのマルチシグアカウントをサポートしています。

NEMのマルチシグトランザクションは、柔軟性を念頭に置いてデザインされました。

他のどんなトランザクション（たとえば、インポータンストランザクション、送付トランザクション、アグリゲート改変トランザクションなど）も、マルチシグトランザクションとして扱うことができます。

ただし、マルチシグトランザクションは、他のマルチシグトランザクションに含まれることはできません（カタパルトでは可能になる？）。

## 4.3.1 アグリゲート改変トランザクション（マルチシグ改変）

**マルチシグアカウントの生成**

一つのアカウントは、アグリゲート改変トランザクションを送ることによって、複数のアカウントの承認が必要なマルチシグアカウントにすることができます。

このアグリゲート改変トランザクションは、共同サインについての情報を持っています。

共同サインする人数の最大数は32です。

**マルチシグアカウントへの追加と削除**

マルチシグアカウントが生成された後、共同サイン者を追加したり削除することができます。

マルチシグトランザクションの中に、アグリゲート改変トランザクションを内包させます。

一つの改変トランザクションで、ひとりまたは複数の共同サイン者を追加できますが、削除は一回につきひとりだけです。

その後、マルチシグアカウントからは、トランザクションを発行できなくなります。

- 共同サイン者を加える時には、現在の共同サイン者全員がトランザクションに共同署名しなければなりません。
- 共同サイン者を削除する場合は、削除されるサイン者以外の全員が共同署名する必要があります。


アグリゲート改変トランザクションの手数料は、改変の内容にかかわらず、一律0.5xemです。

ここで言う「改変」とは、共同サイン者を追加あるいは削除するという意味です。

----

ここでのポイントは、普通のアカウントをマルチシグアカウントにしてしまうと、単体ではトランザクションを発生させることができなくなるということです。また、アグリゲート改変トランザクションによって、あとから共同サイン者を加えたり、削除したりすることができます。これらのトランザクションもm-of-nでのサインが必要になるわけですが、例えば、僕のお葬式の時に、子供と友人との間でマルチシグアカウントを持っていれば、僕の同意がなくても、僕のアカウントをそこから外すことができます。そして僕のアカウントにあったXEMは、相続人である息子のアカウントに無事送ることができたのでした。めでたしめでたし。



## 4.3.2 マルチシグ署名トランザクション　

前回に引き続いて、マルチシグの本体に迫ります。マルチシグ署名トランザクションとマルチシグトランザクションという２つのトランザクションについて説明します。後半では、実際にマルチシグアカウントを作成して、送金をしてみます。

NEMはマルチシグトランザクションに共同サインをするために、マルチシグ署名トランザクションを使います。


マルチシグ署名トランザクションに必要な手数料は0.15XEMです。手数料はマルチシグアカウントから引き落とされ、署名者のアカウントからは引き落とされません。


マルチシグ署名トランザクションは、対応するマルチシグトランザクションがブロックチェーン（ネットワーク？）にすでに存在している場合のみ、ブロックチェーンに書き込まれます。

Orphaned（対応するマルチシグトランザクションが存在しない）マルチシグ署名トランザクションは、ブロックチェーンに書き込まれることはありません。


## 4.3.3 マルチシグトランザクション

すでに述べたように、マルチシグトランザクションにはいろいろなトランザクションを含めることができます。


XEMをマルチシグアカウントから送る場合は、トランスファートランザクションがマルチシグトランザクション化される必要があります。

マルチシグトランザクション化には0.15XEMの手数料がかかります。

この手数料は、通常の送金手数料に上乗せされます。


より詳細に説明するために、例を示します：

まず、マルチシグアカウント（M）が、1000XEMの残高を持っていて、３人の連署者（A, B, C）がいるとします。

このマルチシグアカウントMから別のアカウントXに100XEMを送金してみましょう。


A, B, Cのどの連署者も、100XEMの送金を発生させることができます。

例えば、Bが送金を発生させ、AとCが連署する場合には、以下のステップがトランザクションがブロックチェーンに書き込まれるために必要になります。

1. Bがsignerとしてマルチシグアカウントを設定した100XEMの送金トランザクションを作る。これは未署名の普通の送金トランザクションで良い。
1. Bはこの未署名のトランザクションを、マルチシグトランザクション化する（Walletがやってくれます）。
1. Bは、作ったマルチシグトランザクションにサインをしてから、NEMネットワークに送り出す。
1. AとCは、自分たちが関係するマルチシグトランザクションが存在する事を知らされる（Walletで通知されます）。
1. Aは、未署名の送金トランザクションのハッシュにサインをして、マルチシグ署名トランザクションを作り、ネットワークに送り出す。
1. Cも、同様に未署名の送金トランザクションのハッシュにサインをして、マルチシグ署名トランザクションを送り出す。
1. 全員が未署名の送金トランザクションに署名を終えた時（Bは暗黙的に、A, Cは明示的に）、ようやくこのトランザクションはネットワークに承認されて、MからXに100XEMの送金がなされる。

もしも、AあるいはC（またその両方）が、送金トランザクションの有効期限内にマルチシグ署名トランザクションを送り出さなければ、このマルチシグ送金トランザクションはネットワークによって拒絶され、MからXへの送金は行われないでしょう。


### マルチシグアカウントの作成

では、実際にやってみましょう。まずは、マルチシグアカウントを作るところからです。すでに持っているアカウントをマルチシグにすることもできますが、全財産GOXして伝説のネムロガーになるのはいやなので、新たにmultiというアカウントを作りました。秘密鍵を紙にバックアップしてから、手数料用の1XEMをこのアドレスに送ってスタートです。

![](https://s3-ap-northeast-1.amazonaws.com/nem-social/blog/20000/21000/21900/21942/1556160996%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202019-04-25%2011.29.11.png)

サービス＞マルチシグアカウントへ変換で、nemlogのアカウントと、僕のメインアカウントを連署者のリストとして登録します（連署者が２名だけだと、簡単に乗っ取られてしまうので、実際には３名にすべきです）。また、この時にマルチシグアカウントの秘密鍵が必要です。

すると、次のようなアグリゲート改変トランザクションが、ブロックチェーンに書き込まれます。連署者のアドレスと必要サイン数が登録され、手数料の0.5XEMがマルチシグアカウントから差し引かれます。

![](https://s3-ap-northeast-1.amazonaws.com/nem-social/blog/20000/21000/21900/21942/1556161304%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202019-04-25%2011.33.13.png)

この状態になると、マルチシグアカウントからの送金はできなくなります（受け取りは出来ます）。ですからここで連署者のアドレスを間違って登録したら、ほぼGOX確定です。

![](https://s3-ap-northeast-1.amazonaws.com/nem-social/blog/20000/21000/21900/21942/1556161351%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202019-04-25%2011.35.12.png)


### マルチシグアカウントからの送金

このアカウントからの送金はできないので、マルチシグで送金するには、連署者のアドレスから送金します。

送金の画面に、マルチシグのタブができてますからそこから送金するだけです。いつもお世話になっているSHUさんに送ってみましょう。

![](https://s3-ap-northeast-1.amazonaws.com/nem-social/blog/20000/21000/21900/21942/1556161568%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202019-04-25%2011.40.53.png)


手数料0.15XEMが上乗せされています（手数料はマルチシグアカウントから引かれます）。チャリーンと音がして、マルチシグ送付トランザクションが開始されます（ステップ１〜３）。でも、この状態ではまだブロックチェーンには書き込まれていません。

### 連名での署名

そこで、もう一方の連署アカウント（nemlog用）のウォレットを開いてみると、未承認のトランザクションがあります。

![](https://s3-ap-northeast-1.amazonaws.com/nem-social/blog/20000/21000/21900/21942/1556161731%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202019-04-25%2011.46.48.png)


赤字で「このトランザクションはあな（たの署名が必要です）」と書かれています。ここにパスワードを入れて、マルチシグ署名トランザクションを発生させます（ステップ４〜６）。今回は２つしか連署するアカウントがありませんから、これで終了です。

ここでも、追加の手数料0.15XEMが引かれます。そしてチャリーンです。

![](https://s3-ap-northeast-1.amazonaws.com/nem-social/blog/20000/21000/21900/21942/1556161861%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202019-04-25%2011.51.34.png)

未承認トランザクションが承認になったタイミングで、ブロックチェーンを見てみると、ブロックチェーンにマルチシグ送金トランザクションが書き込まれていました（ステップ７）。Feeは0.05になっており、マルチシグ利用手数料（0.15XEM）は、ブロックチェーンエクスプローラーでは見えないようですが、

![](https://s3-ap-northeast-1.amazonaws.com/nem-social/blog/20000/21000/21900/21942/1556162816%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202019-04-25%2011.50.52.png)

ウォレットではがっつり0.3XEM（2アカウント分）、マルチシグアカウントから引かれていました。手数料の合計は0.35XEMになりました。

![](https://s3-ap-northeast-1.amazonaws.com/nem-social/blog/20000/21000/21900/21942/1556172812%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202019-04-25%2014.49.41.png)


この記事を書くために作ったマルチシグアカウントですが、実用的に使えるように、３番目の連署アカウントを作りました。アグリゲート改変トランザクションを発生させて、新しいアカウントを追加し、2 of 3の連署マルチシグアカウントに昇格させました。

----

このように、マルチシグアカウントの作成と利用には、通常より多くの手間と手数料がかかります。それだけネットワークへの負荷も増えます。しかし、特別なアプリケーションを作ることなく、マルチシグが実現できることにより、非常に高いレベルの安全性が確保されます。また、連署者の共同名義のアカウントとなりますから、企業などの複数での承認作業が必要な場合に、威力を発揮するでしょう。


