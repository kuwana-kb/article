こんばんは、kuwana-kbです。

プログラミングを初めて半年以上経ちますが、データベース周りはあまり勉強していませんでした。
特に設計はからっきしです。
これが影響してクラウドのRDS周りもあまり自信がありません。
なので、弱点を克服すべくデータベース周りの学習をしています。

前回はSQLの入門書籍である「[SQL ゼロから始めるデータベース操作」](https://kuwana-kb.hatenablog.com/entry/2019/04/11/021915)を読みました。
こちらの書籍を通じてある程度SQLの基礎が身についたので、今回はDBの設計について学習しました。
利用した書籍は「<b>達人に学ぶDB設計 徹底指南書</b>」です。

例のごとくまとめていきたいと思います。

[asin:4798124702:detail]


[:contents]

## ざっくりと3行でいうと
   * DB設計初心者にもわかりやすく読みやすい
   * DB設計のアンチパターンを学べる
   * 「一歩進んだ論理設計」の話がおもしろい

## この本の紹介
この本は、DB設計の解説書です。  
内容としては、大まかに以下のような形になっています。

1. RDSと設計の基礎
2. 正規化
3. ER図
4. 正規化とパフォーマンスのトレードオフ
5. DB設計のアンチパターン、グレーノウハウの紹介
6. 一歩進んだ論理設計の紹介

私の場合、DB設計を経験したことがなく、正規化やER図のことがよくわかっていませんでした。
本書では、１からこれらの内容について解説されていて、未経験の私でも最後まで苦もなく読み終えられました。
おそらく以下の要素を満たしていれば問題なく読めると思います。

* 基礎的なSQLを書くことができる
* DBを操作したことがある（業務規模のものだと尚良し）

## この本の３つのポイント

### DB設計初心者にもわかりやすく読みやすい
「この本の紹介」でも述べたとおり、本書は初心者でも理解しやすく読みやすいことがポイントとして挙げられます。

本文は平易な言葉が使われていて、難しい用語には解説がついています。
また、データベース・DB設計の基礎の解説も充実しています。
例えば、DB設計は「システム開発」という大きな枠から解説が始まり、その枠組みの中においてDB設計がどういう位置付けで何故重要なのか、というようなことを理解できます。
この解説があるため、あまり開発工程を経験したことがない方でもスンナリと理解することができると思います。

私としては、やはり正規化・ER図の基礎を身につけられた点が特に良かったと感じます。

### DB設計のアンチパターンを学べる
もうひとつのポイントは、アンチパターン・グレーノウハウについても言及している点でしょう。

アンチパターンなんてそんな現場で起きるのかなーと少し疑問でしたが、
「〇〇のケースの場合、どう設計するのが良いか考えてみてください」といった質問が出てきて解いてみると…
自分の答えが正にそのアンチパターンにあてはまっていたので思わず笑ってしまいました。
こういう浅はかな人間が負債を生み出してしまうのでしょう…反省です。

本書においてアンチパターンは、

* どういう場合にそのアンチパターンが登場するか
* そのアンチパターンの何が悪いのか
* どのような解決策があるか

といった形で説明がなされています。
また、上述に加えて「場合によっては良しとされるグレーノウハウ」においては、
どういったケースで良しとされるかについて言及しています。いずれも納得感のある解説です。

リーダブルコードを読んだ際も感じましたが、やはりアンチパターンから学べることは多いです。
とはいえすべてを覚えることは厳しいので、DB設計をすることになった際に本書のアンチパターンに該当していないか照らし合わせたいと思いました。

### 「一歩進んだ論理設計」の話がおもしろい
本書の最後の方に「一歩進んだ論理設計」という章があります。
ここでは、現状のリレーションナルデータベースの持つ欠点とそれを補う可能性をもつ新しいデータベースモデルの紹介がなされています。
詳細は省きますが、私としては

* 入れ子区間モデル考えた人、天才なのでは…という感嘆
* 未来のデータベースはまた違った構造になる可能性がある…！というわくわく感

を抱いたのでした。

## まとめ
これまでDB設計のことをよくわかっていなかった私ですが、本書を通じて業務領域の視界を広げることができたように思います。

タイトルの「徹底指南書」というワードから難しそうなイメージを受けるかもしれません。
しかし、本書はDBをある程度触ったことがあれば、誰にでもおすすめできる本だと感じました。