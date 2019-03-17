# 「Docker実践ガイド 第二版」 書評

こんばんは、kuwana-kbです。<br>
私のいるチームではDockerを用いた開発を行っているのですが、お恥ずかしいことに私はDockerのことをよくわかっていませんでした。<br>
Dockerに関するタスクはチームの人にお願いしていたのですが、いつまでも任せっきりはチームとして健全じゃないな…という思いもありました。

そこで今回は<b>「Docker実践ガイド 第2版」</b>です。<br>
本書を通じてDockerの基礎を学んで参りましたので書評として投稿したいと思います。

[asin:4295005525:detail]

[:contents]

## ざっくりと3行でいうと
   * Dockerを実戦形式で学べる本
   * Dockerの基礎から近年話題のKubernetesまで網羅されている
   * Webエンジニアだけでなく、IT管理者にもおすすめ

## この本の紹介
この本は、Dockerコンテナの実行環境の構築および管理手法について実践形式で解説した本です。解説の対象は主に以下になります。

* Docker
* Dockerfile
* Docker Community Edition, Docker Enterprise Edition
* Docker管理ツール(Docker Compose, Docker Machine, Private Registry)
* ネットワーキング(Docker Swarm)
* コンテナ用OS(CoreOS, RancherOS)
* オーケストレーションツール（Kubernetes）

では、次に各章のざっくりとした紹介です。

### 第1章 Dockerとは？
この章では、主にDockerの特徴について解説がなされています。物理ホストや仮想化技術との比較を通じて、コンテナ技術の魅力が理解できるでしょう。また、近年のIT業界の歴史というコンテキストから、Dockerの優位性や課題が解説されている点も興味深いです。

### 第2章 Docker導入前の事前準備
この章では、Dockerを導入する前に検討すべき事柄や前提知識に関しての解説がなされています。事前準備というと「個人」としてどういうツールや環境を用意しておくべきかと思うかもしれません。しかし、ここで述べられているのは「エンタープライズ」レベルでの事前準備です。<br>
したがって、この章で解説される内容は、企業内のシステム管理者やIT管理者といった方向けの情報といえるでしょう。

### 第3 ~ 10章 実践
ここからは「この本の紹介」で上げた技術・ツールを実践していきます。すべての項目においてサンプルコードが記載されています。ただし、２つ注意点があります。
1. サンプルコードの実行環境がCentOSであること
2. 物理ホストが3台ほど必要になる場合があること（docker swarm, kubernetes等）
個人で行う場合の代替案については後ほど触れたいと思います。

## この本のポイント
### 2019年初頭のコンテナ技術に関する最新情報が載っている
本書は2019/02/18に出たばかりの本のようで、この書評を書いている時点(2019/03/16)では、コンテナ技術に関する最新情報や今に至るまでの歴史的背景が掲載されています。
この点は、これまでコンテナ技術にあまり触れてこなかった私のような人間にとっては嬉しいことです。

また、解説対象がDocker、コンテナ専用OS、DockerCE/EE（コミュニティ版・エンタープライズ版）、オーケストレーションツールと幅広い点も魅力です。特にDocker CE, EEの解説はあまりネット上には載っていないので、社内オンプレでDockerを構築・運用するような部署（IT管理者やインフラエンジニアでしょうか？）の方には嬉しいのではないでしょうか。

### コンテナの便利さを実感
サンプルコードを実行すると、すぐにコンテナを作れます。
wordpress + mariadbの環境をわずか数コマンドで構築できたときは、その便利さに驚きました。<br>
以下は、私が実際に作ってみた時のツイートです。

[https://twitter.com/kuwana\_kb\_/status/1105168521187622912:embed#DockerのlinkでMediawikiってソフトとMariaDBをつなげた https://t.co/h47EH7QdQg]


### 実行環境に注意が必要
先程も述べましたが、実行環境には注意が必要です。
私の場合はDocker for Mac でサンプルコードを試したのですが、いくつか動作しないものがありました。
これはDockerの仕様上、仕方ない部分のようですが注意が必要です。((おそらくこちらの公式ドキュメントの記載されている内容が原因
[http://docs.docker.jp/v1.12/engine/installation/mac.html#id2:title]))

* mac上のディレクトリをコンテナのディレクトリに対してマウントすることができるのですが、
Docker for macではディレクトリを「File Sharing」で事前に共有しておかないとうまく動かないです。
* systemdを使用するサンプルコードがあるのですが、こちらも注意が必要です。（以下の記事参照）<br>
[https://namionobkup.hatenablog.com/entry/2018/08/07/233157:title]

上記のようにMac（おそらくwindowsも）だとつまづくポイントがあるので、できれば書籍と同じCentOSの環境で学習することをおすすめします。

また、物理ホストを３台使用した例などもあって、個人でやるには難易度が高いものもありました。
こちらについては、私はクラウドインフラで代用しました。
具体的にはkubernetesの環境構築において、GCP(Google Cloud Platform)のGKE(Google Kurbenetes Engine)を使用しました。
（自分でノードの設定等をする必要がないので完全な代替ではないのですが…）

[https://twitter.com/kuwana\_kb\_/status/1106820259447635968:embed#Docker実践ガイド読了kubernetesの章は物理ホストでの解説だったので、GCPに置き換えてやってみた初GKEだったけど、細かな設定はGKE側でマネージドされてるので簡単に構築できた https://t.co/Pg2zea08cY]

GKEを使って思ったのは、マネージドサービスって構築が楽…！ということです。
本書は物理ホストで作業する前提のため、マスターノードやワーカーノードの作業が細かく書かれていたのですが、
そういった作業を一切意識する必要なく簡単にkubernetesを用いた環境構築ができてしまいました。


## まとめ
「Docker実践ガイド 第二版」は解説の幅が広く、IT管理者やサーバーエンジニアの方も満足行く内容だと感じました。
一方でクラウドのマネージドサービスを使用する前提で、とりあえずDockerを使えるようになりたい！という人には、本書は情報過多かもしれません。（もちろん知っておいて損にはなりませんが！）

本書は、Dockerについて実戦形式で学びたい方、Dockerの導入を自社で検討されている方におすすめできる本だと思います。