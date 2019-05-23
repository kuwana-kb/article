# 概要
こんにちは、kuwana-kbです。業務でAWSの Route 53 Resolver を使用する機会がありました。
今回は簡単な解説とCloudFormationのテンプレートを紹介したいと思います。

# Route53Resolverとは
Route 53 Resolverは、AmazonVPCとオンプレ間の名前解決を簡単にしてくれるサービスです。AmazonVPCとオンプレという条件があるため、このサービスを利用するケースとしては、基本的にDirectConnectやVPNを利用しているのが前提になると思います。個人だとあまり使う機会がなさそうな印象ですね。
以下に、公式説明を引用します。
>Amazon VPC とオンプレミス両方のリソースを使用するワークロードを実行しているお客様は、オンプレミスでホストしているプライベート DNS レコードも解決する必要があります。同様に、これらのオンプレミスのリソース側で、AWS でホストしている名前を解決する必要がある場合もあります。そのようなお客様が、名前がホストされている場所に関係なく、Route 53 Resolver のルールとエンドポイントを使用して双方向のクエリ解決を実行できるようになりました。
引用元：[ハイブリッドクラウドの DNS を簡素化する Amazon Route 53 Resolver を発表](https://aws.amazon.com/jp/about-aws/whats-new/2018/11/amazon-route-53-announces-resolver-with-support-for-dns-resolution-over-direct-connect-and-vpn/)

少しイメージしにくいかもしれないので、次に具体的なケースを挙げてみたいと思います。

# 解決したかった課題
![resolver.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/295017/b5033bff-ecc7-a4b2-212f-c9db0b119fa4.png)

今回解決したかった課題は以下になります。

- オンプレのDBである`onpremisesdb.sample.com`に対して、AWSのインスタンスからSQLを投げたい。
-   しかし、`onpremisesdb.sample.com`のDNSレコードはオンプレ上の内部DNSにしか存在せず、Public Network経由では名前解決できない。
※ `onpremisesdb.sample.com`は架空のドメインです。

サービスが「AWSとオンプレの２環境で運用している」や「オンプレからAWSへの移行途中である」といった背景があると、このような課題に直面するかもしれません。この課題を解決するために、Route53 Resolverを使用します。

## 解決方法
今回の解決方法は以下のような流れになっています。
1. VPC内のDNSクエリが`onpremisesdb.sample.com`の場合、Route53 Resolver経由でオンプレのDNS Serverに問い合わせる
2. 「1」の問い合わせで得たIPを元にDBにSQLを投げる

図にすると以下のような流れになります。
「1」の動作でRoute53 Resolverを使用しています。

![resolver3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/295017/e82e7aba-ebf8-6c45-e268-eea896107c98.png)

## CloudFormationでRoute53 Resolverを作ろう
では、実際にCloudFormationでRoute53 Resolverを作ってみましょう。
今回使用するテンプレートで、作成するリソースは以下の4つになります。

* ResolverEndpoint用のセキュリティグループ
* ResolverEndpoint
* ResolverRule
* ResolverRuleAssociation

図にすると以下のような形になります。（VPCとAZは作成されません）
![resolver4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/295017/f0b0fc71-ee7d-c875-5305-344208b6eb8c.png)

各リソースの役割はおそらく以下のようになっています。（ちょっと自信ない）

* ResolverRuleで転送したいドメインと転送先DNSの判断をする
* ResolverEndpointがResolverのエンドポイントとなる
* ResolverRuleAssociationがResolverRuleとVPCを紐付ける（なぜ必要なのかはよくわからない…）
* セキュリティグループでResolverEndpointへのアクセスできる対象を絞る

具体的なパラメータについては、テンプレート内にコメントで記載しています。

## CloudFormationテンプレート

```yml:Resolver.yml
AWSTemplateFormatVersion: '2010-09-09'
Description:
  Create Route53Resolver

Resources:
  # ---------------------------------------------------------------------- #
  # リゾルバエンドポイント用のセキュリティグループ
  # 対象となるVPCからのDNSクエリ(port53)のみ受け付ける
  # ---------------------------------------------------------------------- #
  ResolverEndpointSG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupName: resolver-endpoint-sg 
      GroupDescription: Security group for resolver endpoint. This only allows DNS query from VPC.
      SecurityGroupIngress:
      - CidrIp: 172.0.0.0/16 #endpointへのアクセスを許可するIPを記載する。VPCに限定する場合はVPCのCIDRブロックを書く
        FromPort: 53
        ToPort: 53
        IpProtocol: tcp
      - CidrIp: 172.0.0.0/16
        FromPort: 53
        ToPort: 53
        IpProtocol: udp
      SecurityGroupEgress:
      - IpProtocol: '-1' #アウトバウンドはすべての通信を許可
        CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: resolver-endpoint-sg
      VpcId: vpc-xxxxxxxx #sgと紐付けるVPCのIDを記載する

  # ---------------------------------------------------------------------- #
  # DNSリゾルバのエンドポイント
  # ---------------------------------------------------------------------- #
  ResolverEndpoint:
    Type: AWS::Route53Resolver::ResolverEndpoint
    Properties:
      Direction: OUTBOUND #AWS環境からオンプレ環境へDNSクエリを投げたいのでOUTBOUNDに設定
      IpAddresses:
      - SubnetId: subnet-xxxxxxxx #endpointを設置するsubnetIDを記載する。
      - SubnetId: subnet-yyyyyyyy #リゾルバーの信頼性を高めるため、別々のAZに設置された2つ以上のサブネットIDを指定すること
      Name: resolver-endpoint
      SecurityGroupIds:
      - !Ref ResolverEndpointSG

  # ---------------------------------------------------------------------- #
  # DNSリゾルバのルール
  # DB（onpremisesdb.sample.com）宛のDNSクエリだった場合に、オンプレのDNSにクエリを転送する
  # ---------------------------------------------------------------------- #
  ResolverRule:
    Type: AWS::Route53Resolver::ResolverRule
    Properties:
      DomainName: onpremisesdb.sample.com #転送したいドメイン名を記載する
      Name: test-resolver-rule
      ResolverEndpointId: !Ref ResolverEndpoint
      RuleType: "FORWARD"
      TargetIps:
      - Ip: 10.10.10.10 #オンプレのDNSのIPを記載する
        Port: 53

  # ---------------------------------------------------------------------- #
  # リゾルバとVPCをつなぐためのリソース
  # マネジメントコンソールでは、このリソースは自動で作成される
  # ---------------------------------------------------------------------- #
  ResolverRuleAssociation:
    Type: AWS::Route53Resolver::ResolverRuleAssociation
    Properties:
      Name: resolver-rule-association
      ResolverRuleId: !Ref ResolverRule
      VPCId: vpc-xxxxxxxx

```

Route53 Resolverはマネジメントコンソールから手動で設定することも可能です。ただ、設定をコード化してバージョン管理した方が変更を追いやすく管理しやすいので、今回はCFｎ化しました。テンプレートについては、必要最小限の記述にとどめているため、適宜`Parameters`や`Mappings`等を使って最適化してください。

## Route53 Resolver以外の解決方法について
今回はRoute53 Resolverを用いた解決方法をご紹介しました。しかし、Route53のPrivate Hosted Zoneを用いることでも今回のケースに（一応）対応することができます。

![resolver2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/295017/09c0d994-cd5b-98ca-6f05-0e6d7195a638.png)

どういう方法かというと、オンプレの内部DNSのレコードをAWSのPrivate Hosted Zoneでも保持するという方法です。しかし、この方法だとオンプレの内部DNSレコードに変更がある度にPrivate Hosted Zoneのレコードの修正が必要になります。Route53 Resolverを用いた方が運用コストがかからないので、Route53 Resolverを採用した次第です。

## 参考資料

- [AWS re:Invent 2018: Introduction to Amazon Route 53 Resolver for Hybrid Cloud (NET215)](https://www.youtube.com/watch?v=D1n5kDTWidQ)
- [AWS CloudFormation User Guide](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-resource-route53resolver-resolverrule.html)


