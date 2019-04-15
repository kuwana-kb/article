「プログラムはなぜ動くのか 第2版」を読み終えたのでまとめます。

[asin:4822283151:detail]


## 本のざっくりとした紹介
この本はメモリ、OS、ハードウェアといった低レイヤーの観点から、
プログラムをクリックしてから実際に動作するまでの流れを詳しく解説した本です。  
流れを追っていく中で、エンジニアにとって馴染みのある２進数やポインタ、コンパイルといった仕組みを、
低レイヤーの観点からわかりやすく解説してくれています。

## なぜ読んだか
プログラミングを学んでいく上で避けて通れないものの１つに、「ポインタ」があります。
ポインタでググると「特定のメモリ領域を表現する」といった内容で、C言語を用いた例がよく見受けられます。  
プログラミング初学者の私としては、C言語をそもそも知らないし、ハードウェアの構造もよくわかっていないので、
解説を読んでもしっくり来ることがありませんでした。
本書を通じてポインタの理解を深めることが第一の目的でした。

## 印象に残った内容
読んだ中で特に印象に残った内容を2つ挙げます。

1つめはポインタの型宣言について。
[https://twitter.com/kuwana\_kb\_/status/1089547687483961345:embed#なぜメモリのアドレスであるポインタに対しても型の宣言をするのかと謎だったけど「ポインタに格納されたアドレスから一度に何バイトのデータを読み書きするかを示すため」って記述でなるほどな〜って思った#プログラムはなぜ動くのか]
形式として覚えていた型宣言にもちゃんと理由があったことに驚き。

2つめは低レイヤーでの抽象化について。
[https://twitter.com/kuwana\_kb\_/status/1092445844710682624:embed#OSが提供するシステムコールによってハードウェアを抽象化し、さらに高水準言語によってシステムコールを抽象化する。なのでプログラマーはハードウェア、システムコールを意識する必要がなくなる #プログラムはなぜ動くのか]
プログラミングにおいても様々な手法で抽象化を試みますが、低レイヤーでも抽象化がなされていました。
この抽象化によって我々エンジニアはハードウェアやシステムコールを意識することなくプログラミングに集中できているのですね。

## この本の良いところ
* プログラマ（エンジニア）が読むという前提で書かれているので、必要以上にハードウェアに寄った専門的な単語がなく、読みやすい本でした。
* 解説の中にC言語やアセンブリ言語が登場しますが、言語の解説も入っているので問題なく理解できました。（※）
* 2進数やポインタ、コンパイルといった仕組みを低レイヤーの動作を含めて理解を深めることができました。


本書を通じて当初の目的であったポインタの理解に加え、今まであやふやだった低レイヤーの知識を補完できました。
この日経ソフトウェアさんが出版する「なぜ」シリーズは今回で3冊目ですが、どれも安定してわかりやすいのでおすすめです。

ポインタがよくわからない…という方やプログラムの低レイヤーが気になる方はぜひ読んでみてください。
