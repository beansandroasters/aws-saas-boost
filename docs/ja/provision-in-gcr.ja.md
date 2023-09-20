# GCR での AWS SaaS ブーストのプロビジョニング

## コンテンツ
[イントロダクション](#イントロダクション)\
[利用可能な機能と実装の違い](#利用可能な機能と実装の違い)\
[AWS GCR アカウントに AWS SaaS ブーストをプロビジョニング](#AWS GCR アカウントに AWS SaaS ブーストをプロビジョニング)\
[References](#references)

## イントロダクション
アマゾンウェブサービス中国 (北京) リージョンとアマゾンウェブサービス中国 (寧夏) リージョンは、中国にある 2 つのアマゾンウェブサービスリージョンです。アマゾンウェブサービスは、中国のお客様に最高の体験を提供し、中国の法的および規制上の要件を遵守するために、クラウドサービスの提供に関する適切な通信ライセンスを保有する中国のローカルパートナーと協力しています。北京および隣接地域に拠点を置くアマゾンウェブサービス中国（北京）地域のサービス運営者およびプロバイダーは北京シネットテクノロジー株式会社（Sinnet）であり、寧夏を拠点とするアマゾンウェブサービス（寧夏）地域のサービスオペレーターおよびプロバイダーは寧夏ウエスタンクラウドデータテクノロジー株式会社（NWCD）です。

アマゾンウェブサービスチャイナはアマゾンウェブサービスのグローバルリージョンとは別に運営されているため、機能の可用性と規制要件により、GCR での SaaS Boost プロビジョニングの経験は他に類を見ません。このドキュメントでは、GCR での AWS SaaS Boost のプロビジョニングに関するガイダンスを提供します。

## 利用可能な機能と実装の違い
1. システムユーザーサービスは以下の点で異なります。
    - Amazon Cognito ユーザープールは現在 GCR リージョンではご利用いただけません。
    - Keycloak（オープンソースのIDおよびアクセス管理）は、別の ID プロバイダーとして提供されています。
2. 管理用ウェブ UI は以下の点で異なります。
    - デフォルトの CloudFront ドメイン `*.cloudfront.cn` を使用してコンテンツを配信することはできません。CloudFront ディストリビューションに代替ドメイン名 (CNAME とも呼ばれる) を追加し、そのドメイン名をコンテンツの URL に使用する必要があります。また、[ICP 登録] (https://www.amazonaws.cn/en/about-aws/china/#ICP_in_China) も必要です。さらに、グローバル CloudFront サービスと同様に、HTTPS 経由でコンテンツを配信するには、代替ドメイン名に SSL/TLS 証明書を使用する必要があります。
    - 中国リージョンの Amazon CloudFront は、現在 AWS Certificate Manager をサポートしていません。別の第三者認証局 (CA) から SSL/TLS 証明書を取得し、IAM 証明書ストアにアップロードする必要があります。
3. Amazon SES は、以下の点で異なります。
    - Amazon SES は GCR リージョンではご利用いただけません。
4. Amazon QuickSight は、以下の点で異なります。
    - Amazon QuickSight は GCR リージョンではご利用いただけません。

## AWS GCR アカウントに AWS SaaS ブーストをプロビジョニング
インストールプロセスを準備するには、必要に応じて次の手順を実行します。
1. ドメインがあり、ドメインは [ICP 登録済み](https://www.amazonaws.cn/en/about-aws/china/#ICP_in_China)。
2. Keycloakのインストールと管理用 Web UIには、ルートドメイン名と個別のドメイン名が必要です。その一環として、Keycloak ドメイン用の ACM 証明書と、管理用ウェブ UI ドメインをカバーする IAM の証明書が 1 つ必要です。ワイルドカード証明書 (例:*.example.com) または特定の証明書 (例:keycloak.example.com)。

    - ドメインのパブリックホストゾーンを AWS Route53 に作成する必要があります。
    - [証明書を ACM へ](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request-public.html)リクエストまたはアップロードします。
    - [証明書を IAM へ](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cnames-and-https-procedures.html)アップロードします。

インストールプロセスを開始するには、次の手順を実行します。
1. ターミナルウィンドウから、AWS SaaS Boost (aws-saas-boost) をダウンロードしたディレクトリに移動します。
2. インストールコマンドを起動します。
      - Linux または OSX 上で実行している場合は、`sh /install.sh` を実行してください。
      - Windows 上で実行している場合は、 `powershell .\install.ps1` を実行してください。
3. 新規インストールのオプションを選択します。
4. AWS SaaS Boost ディレクトリへのフルパスを入力します (現在のディレクトリは Enter キーを押します)。 /\<mydir\>/aws-saas-boost
5. この SaaS Boost 環境の名前 (dev, QA, test, sandbox など) を入力します。
6. 初期一時パスワードを受け取る AWS SaaS Boost 管理者のメールアドレスを入力します。AWS GCR リージョンでは、管理者インストールの基本パスワードは SecretsManager のシークレット「sb-{env}-admin」に保存されます
7. システムユーザーが使用するIDプロバイダーとして `Keycloak` と入力します。
8. コントロールプレーン ID プロバイダーのドメイン名を入力します。
9. Keycloak ドメインには Route53 ホストゾーンと ACM 証明書を選択します。
10. SaaS Boost 管理用ウェブコンソールのドメイン名を入力します。
11. SaaS ブースト管理 UI ドメインの Route53 ホストゾーンと IAM 証明書を選択します。
12. AWS SaaS Boost のメトリクス機能と分析機能をインストールするかどうかを指定してください。これは***オプション***で、[Redshift](https://aws.amazon.com/redshift) クラスターをプロビジョニングします。
      - **N** と入力できます。QuickSight は現在 GCR では使用できません。
13. アプリケーションが Windows ベースで共有ファイルシステムを必要とする場合は、[Amazon FSx for Windows File Server](https://aws.amazon.com/fsx/windows/) または  [Amazon FSx for NetApp ONTAP](https://aws.amazon.com/fsx/netapp-ontap/) をサポートするために  [Managed Active Directory](https://aws.amazon.com/directoryservice/) をデプロイする必要があります。必要に応じて [y] または [n] を選択します。
14. インストールの設定を確認してください。AWS アカウント番号と AWS SaaS Boost をインストールしようとしている AWS リージョンを再確認してください。**y** を入力して続行するか、**n** を入力して値を再入力または修正します。

インストールプロセスでは、すべてのリソースの設定とプロビジョニングに 30 ～ 45 分かかります (選択したオプションによって異なります)。インストールプロセスの詳細なログは、**saas-boost-install.log** に保存されます。

## レファレンス
- [Amazon Web Services in China](https://www.amazonaws.cn/en/about-aws/china/)
- [Amazon Cognito](https://docs.amazonaws.cn/en_us/aws/latest/userguide/cognito.html)
- [Amazon Cloudfront](https://docs.amazonaws.cn/en_us/aws/latest/userguide/cloudfront.html)