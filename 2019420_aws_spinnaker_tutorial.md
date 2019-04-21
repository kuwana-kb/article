## 概要
![スクリーンショット 2019-04-19 21.20.43.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/295017/9fa7c0e9-5e4d-a90c-3837-327924b8eb4b.png)

<b>Spinnaker</b>とは、NetflixとGoogleを中心に開発されている、マルチクラウドに対応した継続的デリバリー(CD)ツールです。カナリアリリースやB/Gデプロイ、承認制デプロイといった仕組みをカンタンに実現できるそうです。業務で調査する機会を頂いたので試してみました。

今回はAWS上でSpinnakerを試してみたいという方向けに、Spinnakerサーバを構築する手順を解説します。[AWSではSpinnakerのQuickstart](https://aws.amazon.com/jp/quickstart/architecture/spinnaker/)が用意されていますが、あまり保守されておらず手順通りに実行しても正常に動作しません（少なくとも私が確認した限りでは…）。そこで、Spinnaker公式のドキュメント「[Set up Spinnaker](https://www.spinnaker.io/setup/)」を利用します。しかし、こちらもひとクセある&ネット上にAWS×Spinnakerの事例がほとんどない状態で苦労します（しました）。これから試したいという方がスムーズにできるように解説していきます。

構築にあたって、[Halyard](https://github.com/spinnaker/halyard)というSpinnaker用のCLIツールを使います。Spinnakerは複数のコンポーネントで構築されているのですが、Halyardを通じてそれらの設定やデプロイを行うことができます。

この解説書を実行すると、最終的に以下のリソースが立ち上がります。AWSの操作方法さえわかれば割と簡単に構築できるので是非試してみてください。

![spinnaker.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/295017/f02dfeb0-be8c-2c2f-0b96-f083cb677cfa.png)

## 1章 Spinnaker用にAWS環境を構築する
この章では、Spinnakerを動かすための基盤を作っていきます。具体的には、VPCやサブネット、IAM Roleといったリソースで構成されています。これらのリソースはSpinnakerの公式が用意しているCloudFormationのテンプレートを用いて作ります。テンプレートは「[Spinnaker - Amazon EC2](https://www.spinnaker.io/setup/install/providers/aws/aws-ec2/)」の`2. Download the template locally to your workstation.`というところから2つ取得します。

取得したテンプレートである「Managing.yml」と「[Managed.yml]」をざっくりと説明します。

* `Managing.yml` : Spinnakerサーバを動作させるための基盤および認証要素を作成する
* `Managed.yml` : Spinnakerサーバの管理対象となるリソースの認証要素を作成する

これらのテンプレートを実行すると以下の赤枠部分のリソースが作られることになります。ちなみにこの手順では、`米国西部（オレゴン us-west-2）`で作成します。

<img width="1002" alt="スクリーンショット_2019-04-18_21_30_33.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/295017/f55ae498-c4e8-ac34-59c6-cc3570ec77fa.png">

引用元：[Spinnaker - Set up AWS Overview](https://www.spinnaker.io/setup/install/providers/aws/)

### 1-1.Managing.ymlのスタックを構築します
`Managing.yml`を使ってスタックを作っていきます。パラメータは以下のように設定しましょう。

|Key|Value|備考|
|-----|-----|-----|
|スタックの名前|任意|
|EksClusterName|None|SpinnakerサーバにEKSを利用する場合はリソース名を入力します。今回は必要とする知識が少ないEC2を選択するので、Noneにします。|
|SpinnakerPublicSubnet1CIDR|10.100.10.0/24|
|SpinnakerPublicSubnet2CIDR|10.100.11.0/24|
|SpinnakerVPCCIDR|10.100.0.0/16|
|UseAccessKeyForAuthentication|false|SpinnakerサーバからAWS上のリソースへの認証方法を決めます。trueはアクセスキーを用います。falseはIAM Roleを用います。|

##### UserAccessKeyForAuthenticationについて
AWSの認証に関するベストプラクティスでは、アクセスキーではなく IAM Roleの使用が推奨されています。
これに倣って、今回はIAM Role(false)を使用します。実際アクセスキーよりもIAM Roleの方が管理の手間が省けて楽です。
アクセスキーのパターンも試しましたが、CFnテンプレートに不備があるため修正が必要でした。（テンプレートの分岐処理がちゃんとできていれば修正作業は不要なはずなんですが、まだそこまで整備されていないようです。）
cf.[AWS アクセスキーを管理するためのベストプラクティス](https://docs.aws.amazon.com/ja_jp/general/latest/gr/aws-access-keys-best-practices.html)

設定を終えたら、スタックを作成しましょう。
なお、このスタックではVPCを作成することになるため、作業するアカウントによっては権限が不足でエラーになる可能性があります。
その場合は、アカウントを切り替えるかVPC作成権限のついたcfnのIAM Roleで実行するようにしてください。

<details><summary>tips:スタック作成時に、BaseIAMRoleの重複によるエラーが起きた場合</summary><div>

既にAWS内に`BaseIAMRole`がある場合、リソースが重複してCfnの実行が止まってしまうためエラーになっています。
まず、`Managing.yml`の`BaseIAMRole`をコメントアウトします。 

```yml:Managing.yml
 Resources:

  # BaseIAMRole:
  #   Properties:
  #     RoleName: BaseIAMRole
  #     AssumeRolePolicyDocument:
  #       Statement:
  #         - Action:
  #             - sts:AssumeRole
  #           Effect: Allow
  #           Principal:
  #             Service:
  #               - ec2.amazonaws.com
  #       Version: '2012-10-17'
  #     Path: /
  #   Type: AWS::IAM::Role
```
続いてBaseInstanceProfileのRolesをRole名で指定します。(94行目)
これは、既にAWSを上にBaseIAMRoleが存在するため、テンプレートからではなくAWS上のリソース名で指定する必要があるためです。

```yml:Managing.yml
  # Creates Instance Profile to be used by any APP created by Spinnaker. Spinnaker has passRole access only to this instance Profile
  BaseInstanceProfile:
      DependsOn: SpinnakerAuthRole
      Condition: CreateEc2Role
      Properties:
        InstanceProfileName: BaseInstanceProfile
        Path: /
        Roles:
          - BaseIAMRole
      Type: AWS::IAM::InstanceProfile
```
</div></details>

### 1-2. Managed.ymlのスタックを構築します
次に`Managed.yml`からスタックを作っていきます。
以下参考に入力してください。

|Key|Value|備考|
|-----|-----|-----|
|スタックの名前|任意|cfn上での識別するためのものなのであまり深く考えなくてOKです|
|AuthArn|例：arn:aws:iam::1234567890:role/SpinnakerAuthRole|Managing テンプレートの出力からコピー|
|ManagingAccountId|例：1234567890|Managing テンプレートの出力からコピー|

なお、ここで入力するパラメータは`Managing.yml`で作ったスタックの出力欄からコピペしてきましょう。
![スクリーンショット_2019-04-19_21_41_24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/295017/4d163080-f09a-bec5-4707-010dfaddcb64.png)


値の入力が完了したら、スタックの作成をします。以上でCloudFormation上に「Managing」と「Managed」の２つのスタックが構築されました。


## 2章 Spinnaker用のEC2インスタンスを設定する

![スクリーンショット 2019-04-19 21.43.04.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/295017/833c32ce-5939-7f69-683f-533ea1ef0727.png)

### 2-1.Spinnaker用のインスタンスを作ります
今回は、インスタンスとしてEC2を選択します。
細かい設定は下の方にまとめますが、ポイントをいくつか説明します。

<b>Spinnaker用EC2インスタンスの作成ポイント</b>

* AMIはUbuntuの14.04か16.04にする。Spinnakerの推奨OS。
* インスタンスタイプは、T2系<b>非推奨</b>。スペックが低いと途中で処理落ちする。今回はm5でやる。
* IAM RoleはManagingで作ったものを適用する。
ちゃんと設定しないとSpinnakerサーバがAWSのリソースにアクセスできない。

以下の設定を終えたらインスタンスを作成しましょう。

<b>AMIの選択</b>

|項目|値|備考|
|-----|-----|-----|
|AMI|Ubuntu Server 16.04 LTS (HVM), SSD Volume Type|Spinnakerの[Ubuntuの推奨バージョンは 14.04 or 16.04](https://www.spinnaker.io/setup/install/)です。|

<b>インスタンスタイプの選択</b>


|項目|値|備考|
|-----|-----|-----|
|インスタンスタイプ|m5.xlarge|インスタンスのスペックが貧弱すぎると、処理の途中で止まってしまいます。|

<b>インスタンスの設定</b>

|項目|値|備考|
|-----|-----|-----|
|ネットワーク|SpinnakerVPC||
|サブネット|SpinnakerVPC.external.us-west-2a|なぜかサブネットが２つともパブリックなのでどちらでもOK。<br>本番運用なら踏み台を挟んで、このインスタンスはプライベートにおいたほうが良いです。今回は手間を省くためパブリックに立てます。|
|自動割当パブリックIP|有効|今回は直接sshしたいので有効化します。セキュリティはSGでカバーします。|
|IAM ロール|test-spinnaker-managing-SpinnakerInstanceProfile-XXXXXX|Managingテンプレートで作成したロールです。ここで指定する物理IDはCFnのリソース欄から特定できます|
<b>セキュリティグループ</b>


|タイプ|プロトコル|ポート範囲|ソース|備考|
|-----|-----|-----|-----|-----|
|SSHTCP|22|カスタム $your_ip_address|任意|$your_ip_addressに任意のIPを入れてください。<br>これを設定しないとどこからでもsshし放題になるので注意してください。|

<b>キーペア</b>

「既存のキーペアの選択」、「新しいキーペアの選択」どちらでもOKです。
キーペアを登録していない場合は後者を選択しましょう。

次はSpinnakerを導入します。

## 3章 Spinnakerの導入
![スクリーンショット 2019-04-19 21.44.56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/295017/8705e1b6-53eb-f559-d243-eb87d94fd062.png)

いよいよSpinnakerを導入していきます。
ここはつまづくことが多いので設定に注意しながらやりましょう。

### 3-1.SpinnakerサーバにSSHする
以下のコマンドでSSHします。

```Bash:Client
$ ssh -L 9000:localhost:9000 -L 8084:localhost:8084 -L 8087:localhost:8087  ubuntu@xxx.xxx.xxx.xxx -i $key_pair 
```

9000や8084といった値はフォワーディングするポートを指します。ポートフォワーディングによって、クライアントとSpinnakerを構成するコンポーネントをつなぐことで、クライアントのlocalhostからSpinnakerのUIに接続することができるようになります。また、さきほど登録したキーペアの指定を忘れないようにしましょう。

### 3-2.Spinnakerの導入
ここからSpinnakerを導入していきます。
`hal` というコマンドがSpinnakerのコマンドラインツール、Halyardになります。
以下にterminal上の操作をまとめました。

```Bash:ubuntu@SpinnakerServer
## Halyardをインストールする
## @see https://www.spinnaker.io/setup/install/halyard/
$ curl -O $ https://raw.githubusercontent.com/spinnaker/halyard/master/install/debian/InstallHalyard.sh
$ sudo bash InstallHalyard.sh
#ユーザーを選択する必要がでるので、ubuntuと入力

## halのバージョンを確認
$ hal -v


## AWSの設定をするため、設定を変数に入れる
$ REGION=us-west-2 #使っているリージョン名に応じて変えてください
$ AWS_ACCOUNT_NAME=my-aws-account #これは任意の名前でOKです。SpinnakerのUI上にAWSのアカウント名として表示されます。
$ ACCOUNT_ID=1234567890　#AWSアカウントのIDを入れてください

## HalyardのクラウドプロバイダーとしてAWSを設定する
## @see https://www.spinnaker.io/setup/install/providers/aws/aws-ec2/
$ hal config provider aws account add $AWS_ACCOUNT_NAME \
    --account-id ${ACCOUNT_ID} \
    --assume-role role/spinnakerManaged

$ hal config provider aws enable


## Halyardのデプロイタイプをlocaldebianに設定
## see https://www.spinnaker.io/setup/install/environment/
$ hal config deploy edit --type localdebian


## Spinnakerのデータを保存する永続化ストレージとしてS3を設定
## S3上に「spin-12367957-d6a2-4bfe5-8f2e4-a84drgdb8549e28」といった形式でバケットが作成される
## see https://www.spinnaker.io/setup/install/storage/s3/
$ hal config storage s3 edit --region $REGION

$ hal config storage edit --type s3


## 設定した内容でSpinnakerをデプロイ
## see https://www.spinnaker.io/setup/install/deploy/

## Halyardのバージョンを確認して設定する。今回は最新の1.13.4にした
$ hal version list
$ hal config version edit --version 1.13.4

## 設定した内容でSpinnakerをデプロイ
$ sudo hal deploy apply
$ hal deploy connect

## デプロイはここまでで一旦完了
## ただし、この段階だと正常に動かないので以下の対応を行う

## redisを起動する。また、再起動時に自動で起動するように設定する
## redisが起動していないとSpinnakerの中のgateというコンポーネントにエラーが生じるため、この対応を行う
$ sudo systemctl enable redis-server.service
$ sudo systemctl start redis-server.service

## spinnakerを再起動。また、自動で起動するように設定する
## これで全てのコンポーネントが正常に立ち上がる
$ sudo systemctl enable spinnaker.service
$ sudo systemctl restart spinnaker.service
```

### 3-3.Spinnakerの立ち上がりを確認する
先程のterminalの操作でSpinnakerが立ち上がったはずです。
しかし、Spinnakerの立ち上がりは少し遅いので、netstatを用いて確認しましょう。

```Bash:ubuntu@SpinnakerServer
$ netstat -tulpn
```

まず、立ち上げ直後の状況は以下のような感じです。127.0.0.1:XXXXでLISTENしてるプロセスが4つしかありません。

![スクリーンショット_2019-04-18_15_47_43.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/295017/e91dd85d-2f14-260e-0e87-66ed654e6f85.png)

2,3分待つと全て立ち上がります。合計で9つになりました。

![スクリーンショット_2019-04-18_15_53_29.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/295017/c288510e-2a4f-65e1-953e-101c74224a3d.png)

参考までに、上記のポートがSpinnaker上のどのコンポーネントに対応しているかまとめました。解説欄に「注意」とあるのは、調査の段階でよくエラーの原因となっていたコンポーネントです。あまり触れていないコンポーネントは[公式のアーキテクチャ解説](https://www.spinnaker.io/reference/architecture/)をざっくり意訳してます。

|コンポーネント|ポート|解説|
|-----|-----|-----|
|Deck|9000|<b>注意</b><br>ブラウザベースのUIを提供する。<br>これが正常に起動しないと、Spinnakerの画面が表示されない。|
|Igor|8088|CI系の動作を管理する。<br>例えばJenkinsやTravisCI経由でパイプラインを動作させるといった役割を担う。|
|Echo|8089|通知系を管理する。<br>具体的には、slack通知やメールの送信、Githubのwebhookを受け取る。|
|Clouddriver|7002|クラウドプロバイダ(AWS、GCPなど)へのリクエストや<br>デプロイされたリソースのインデクス・キャッシュ周りを管理する。|
|Front50|8080|<b>注意</b><br>Spinnaker上のデータを管理する。<br>様々なストレージとのインターフェース的な役割をする。<br>ストレージの設定不備でエラーを起こしがち。|
|Orca|8083|オーケストレーションエンジン。<br>特定のオペレーションやパイプラインを管理する。|
|Gate	|8084	|<b>注意</b><br>SpinnakerのAPI Gateway。<br>これが正常立ち上がらないとと何もできない。<br>こちらもエラーの主要因。|
|Rosco|8087|クラウドプロバイダに応じたマシンイメージ（AWSならAMI）を作成する。|
|Redis|6379|<b>注意</b><br>このRedisが起動していないとGate が正常に立ち上がらない。|

### 3-4.Spinnakerにブラウザからアクセスする
既にSSHでポートフォワーディングしているので、localhostからSpinnakerを確認することができるはずです。 `http://localhost:9000/` にアクセスしてみましょう。以下のような画面が表示されるはずです。

![スクリーンショット 2019-04-18 22.17.57.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/295017/31412e0f-4e6a-0405-7784-7f264a4e21db.png)

以上で、Spinnakerの構築は完了です。

## 課題
一通り構築を終えて感じた課題をまとめます

#### トラブルシューティングが結構大変
* まだ開発段階のため、頻繁に仕様が変わる
* GCPの事例は見つかるが、AWSの事例はあまりない
* 公式ドキュメントが英語しかないので英語は避けて通れない

こういったことから不具合の原因究明が結構大変でした。
特に最初の方はこの辺のコストがかかることを覚悟する必要がありそうです。

#### Spinnaker公式のCFnテンプレートに問題がある
今回利用したテンプレートは、リソースの設計が誤っていてセキュリティ的にも微妙な構成です。
ex.パブリックサブネット×2の構成、踏み台を返さず直接インスタンスにsshする構成
お試しであれば問題ないですが、本番導入する場合はテンプレートを自作する必要があると思います。

次回はSpinnakerのパイプライン機能を用いて、 Continuous Delivery をどのように実現できるか試してみたいと思います。
まだSpinnakerを触り始めたばかりなので、知見を共有していただけると嬉しいです。

### おまけ アクセスキーを用いる場合の注意点
「1章 Spinnaker用にAWS環境を構築する」で IAM Roleを認証方法に選びましたが、アクセスキーを選択する場合の注意点も念の為記載します。

アクセスキーを選択した上で`managing.yml`を実行しようとするとエラーになります。アクセスキーを使用する場合`Resources`の`SpinnakerInstanceProfile`は不要になるのですが、その場合でも`Outputs`としてInstanceProfileのArnを出力しようとするのが原因です。これを避けるためには、`Outputs`の`SpinnakerInstanceProfileArn`をコメントアウトする必要があります。

```
  EksClusterName:
      Condition: SupportEKS
      Value: !Ref EksClusterName

   # SpinnakerInstanceProfileArn:
   #   Value: !GetAtt SpinnakerInstanceProfile.Arn

  VpcId:
      Value: !Ref SpinnakerVPC
```

