# AWS SaaS Boost 開始方法ガイド

## Contents
[イントロダクション](#イントロダクション)\
[対象とする体験](#対象とする体験)\
[ステップ 1 - ツールのセットアップ](#ステップ-1---ツールのセットアップ)\
[ステップ 2 - AWS SaaS Boost レポジトリのクローン](#ステップ-2---AWS-SaaS-Boost レポジトリのクローン)\
[ステップ 3 - AWS アカウントに AWS SaaS Boost をプロビジョニング](#ステップ-3---AWS-アカウントに-AWS-SaaS-Boost-をプロビジョニング)\
[ステップ 4 - AWS SaaS Boost にログイン](#ステップ-4---AWS-SaaS-Boost-にログイ)\
[ステップ 5 - ティアとアプリケーション設定の構成](#ステップ-5---ティアとアプリケーション設定の構成)\
[ステップ 6 - アプリケーションサービスの更新](#ステップ-6---アプリケーションサービスの更新)\
[ステップ 7 - (オプション) サンプルアプリケーショのデプロイ](#ステップ-7---(オプション) サンプルアプリケーショのデプロイ)\
[アプリケーションと AWS SaaS Boost の紐付け](#アプリケーションと-AWS-SaaS-Boost-の紐付け)

## イントロダクション
このドキュメントでは、AWS SaaS Boost でワークロードをインストール、設定、実行開始するための基本的な手順について説明します。システムをより包括的に把握するには、他の AWS SaaS Boost ドキュメントを参照してください。

このドキュメントでは、AWS SaaS Boost をセットアップする手順の概要を説明していますが、ユーザー体験や基盤となるテクノロジーを深く掘り下げることを意図したものではありません。これらの詳細は [ユーザーガイド](user-guide.ja.md) と [開発者ガイド](developer-guide.ja.md) で説明されています。

## 対象とする体験
AWS SaaS Boost の設定に必要なステップを掘り下げる前に、環境の基本要素を見て、AWS SaaS Boost で何が可能になるのかを理解しておきましょう。以下の図は、AWS SaaS Boost 体験の主要コンポーネントをセットアップフローの順に示しています。

![セットアップロー](../images/gs-install-flow.png?raw=true "Setup Flow")

以下は各ステップの内訳です。
1. インストールプロセスに必要なツールを設定
2. AWS SaaS ブーストリポジトリのクローンを作成
3. AWS アカウントに AWS SaaS ブーストをプロビジョニング
4. AWS SaaS Boost 管理ウェブアプリケーションにアクセス
5. 要件に合わせて階層とアプリケーション設定を構成
6. アプリケーションの各サービスの Docker イメージをアップロード

この段階で、環境内の変動要素がすべて整い、テナントのオンボーディングを開始できるようになりました。AWS SaaS Boost アプリケーションを使用して新しいテナントをオンボーディングすることも、アプリケーションから API 呼び出しを行って新しいテナントのオンボーディングをトリガーすることもできます。

最後に考慮すべき点は、アプリケーションの更新プロセスです。アプリケーションに変更が加えられたら、最新バージョンを AWS SaaS Boost にアップロードできます。新しいバージョンがアップロードされるたびに、システム内のすべてのテナントに自動的にデプロイされます。AWS SaaS Boost により、SaaS ビジネスとして運用する準備が整いました。これには、AWS SaaS Boost 環境に組み込まれているオペレーションおよび管理ツールが含まれます。

AWS SaaS Boost のテクノロジーと経験について理解できたところで、次は AWS SaaS Boost の設定に関する詳細を見てみましょう。

## ステップ 1 - ツールのセットアップ
AWS SaaS Boost は、インストールプロセスにいくつかのテクノロジーを使用します。ご使用のオペレーティングシステムに以下の前提条件をそれぞれインストールし、設定します (まだインストールしていない場合)。
- Java 11 [Amazon Corretto 11](https://docs.aws.amazon.com/corretto/latest/corretto-11-ug/downloads-list.html)
- [Apache Maven](https://maven.apache.org/download.cgi) (see [Installation Instructions](https://maven.apache.org/install.html))
- [AWS Command Line Interface version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- [Git](https://git-scm.com/downloads)

前提条件をすべてローカルマシンにインストールできない、またはインストールしたくない場合は、AWS Cloud9 インスタンスを使用して AWS SaaS Boost をインストールできます。[AWS Cloud9 を使用してインストール](./install-using-cloud9.ja.md) の手順に従ってください。その後、 [ステップ 3](#ステップ-3---AWS アカウントに AWS SaaS Boost をプロビジョニング) に進んでください。

## ステップ 2 - AWS SaaS Boost レポジトリのクローン
ツールが用意できたので、AWS SaaS Boost のコードとインストールスクリプトをダウンロードできるようになりました。ターミナルウィンドウを開き、AWS SaaS Boost アセットを保存するディレクトリに移動します。以下のコマンドを実行して AWS SaaS Boost リポジトリをクローンします。
`git clone https://github.com/awslabs/aws-saas-boost ./aws-saas-boost`

## ステップ 3 - AWS アカウントに AWS SaaS Boost をプロビジョニング
コードの準備ができたので、所有および管理している AWS アカウントに AWS SaaS Boost をインストールする必要があります。インストールプロセスでは、AWS SaaS Boost の設定に必要なすべてのリソースをプロビジョニングして設定するスクリプトが実行されます。インストールを実行するシステムには、4 GB 以上のメモリと高速インターネットアクセスが必要です。

インストールを実行する前に、[AWS CLI をセットアップしてください](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html):
1. 完全な管理者権限を持つ IAM ユーザーを AWS アカウントに設定します。
2. AWS アクセスキーとデフォルトリージョンを使用して AWS CLI 認証情報を設定します。

AWS **中国リージョン (北京/寧夏) **のいずれかに SaaS Boost をインストールする場合は、[GCR での AWS SaaS Boost のプロビジョニング](/provision-in-gcr.ja.md)または [スタートガイドの中国語版](./getting-started.cn.md)の指示に従ってください。

インストールプロセスを開始するには、次の手順を実行します。
1. ターミナルウィンドウから、AWS SaaS ブースト (aws-saas-boost) をダウンロードしたディレクトリに移動します。
2. インストールコマンドを起動します。
      - Linux または OSX 上で実行している場合は、以下を実行します。: `sh ./install.sh`
      - Windows で実行している場合は、以下を実行します。: `powershell .\install.ps1`
3. 新規インストールのオプションを選択します。
4. AWS SaaS Boost ディレクトリへのフルパスを入力します (現在のディレクトリは Enter キーを押します)。: /\<mydir\>/aws-saas-boost.
5. この SaaS Boost 環境の名前 (dev, QA, test, sandbox など) を入力します。
6. 初期一時パスワードを受け取る AWS SaaS Boost 管理者のメールアドレスを入力します。
7. SaaS Boost 管理用ウェブコンソールの ID プロバイダーを選択します。「Cognito」または「Keycloak」のいずれかを入力します (Cognito の場合は [Enter] を押します)。[Keycloak](https://www.keycloak.org/) を使用するには、検証済みの TLS (SSL) 証明書付きの登録済みドメイン名、既存の Route53 ホストゾーン、および動作中の DNS 解決が必要です。
      - `Keycloak`を選択すると、Keycloakのインストールに使用するカスタムドメイン名の入力を求められます。- SaaS Boostは既存のRoute53ホストゾーンを検索し、選択できるリストを提供します。既存のカスタムドメインのホストゾーンの番号を入力して選択します。
      - SaaS Boostは既存の ACM 証明書を検索し、選択可能なリストを提供します。既存のカスタムドメインの証明書の番号を入力して選択します。
8. SaaS Boost 管理 Web コンソールにカスタムドメイン名を使用するかどうかを選択します。カスタムドメイン名を使用するには、検証済みの TLS (SSL) 証明書、既存の Route53 ホストゾーン、および動作中の DNS 解決が必要です。 
      - `y` を入力すると、SaaS Boost 管理Webコンソールのインストールに使用するカスタムドメイン名の入力を求められます。（システムユーザー IdP とし てKeycloak を使用している場合、このドメインは Keycloak ドメインと異なる必要があることに注意してください）。
      - SaaS Boost は既存の Route53 ホストゾーンを検索し、選択可能なリストを提供します。既存のカスタムドメインのホストゾーンの番号を入力して選択します。
      - SaaS Boost は既存の ACM 証明書を検索し、選択可能なリストを提供します。既存のカスタムドメインの証明書の番号を入力して選択します。
9. オプションの分析パイプラインとデータウェアハウスをインストールするかどうかを選択します。これは***オプション***で、[Redshift](https://aws.amazon.com/redshift) クラスターをプロビジョニングします。
      - `y` と入力すると、[QuickSight](https://aws.amazon.com/quicksight/) をセットアップするように求められます。Quicksight を使用するには、[https://docs.aws.amazon.com/quicksight/latest/user/signing-up.html](https://docs.aws.amazon.com/quicksight/latest/user/signing-up.html) の手順に従って、AWS アカウントのスタンダードまたはエンタープライズ Quicksight アカウントに _登録済みである必要があります_。
10. インストールの設定を確認してください。AWS アカウント番号と AWS SaaS Boost をインストールしようとしている AWS リージョンを再確認してください。続行するには `y` を、キャンセルするには `n` を入力します。

インストールプロセスでは、すべてのリソースの設定とプロビジョニングに 30 ～ 45 分かかります (選択したオプションによって異なります)。インストールプロセスの詳細なログは、**saas-boost-install.log** に保存されます。

## ステップ 4 - AWS SaaS Boost にログイン
インストールプロセスの一環として、インストールプロセス中に指定したメールアドレスにメッセージが届きます。このメッセージには、AWS SaaS Boost 管理アプリケーションへの URL へのリンクが含まれています。メッセージは次のように表示されます。

![ウェルカムメール](images/gs-welcome-email.png?raw=true "Welcome Email")

ここに表示されている一時パスワードをコピーします。インストールスクリプトが完了すると、AWS SaaS Boost 管理アプリケーションの同じ URL も出力されます。Web ブラウザを使用して管理アプリケーションに移動します。次のログイン情報が表示されます。

![ログイン画面](images/gs-login.png?raw=true "Login Screen")

ユーザー名に `admin` と入力し、前述のメールで配信された一時パスワードを入力します。仮パスワードでログインしているため、アカウントの新しいパスワードを入力するよう求められます。画面は次のように表示されます。

![パスワード変更](images/gs-change-password.png?raw=true "Change Password")

## ステップ 5 - ティアとアプリケーション設定の構成
AWS SaaS Boost 管理アプリケーションにログインしたら、最初に行う必要があるのは、アプリケーションのニーズに合わせて環境を設定することです。

AWS SaaS Boost では、_tiers_ に基づくアプリケーション設定の構成をサポートしています。ティアにより、SaaS サービスをさまざまなティアセグメント向けにパッケージ化できます。たとえば、スタンダードティアの他に、トライアルティアやプレミアムティアがある場合があります。AWS SaaS Boost は、お客様に代わって ***default*** ティアを作成します。デフォルトティアの名前を変更したり、追加のティアを作成したりする場合は、画面左側のナビゲーションから**階層**を選択します。次のようなページが表示されます。
![アプリケーションセットアップ](images/gs-tiers.png?raw=true "Tiers")

**Create Tier** ボタンをクリックして新しいティアを作成するか、テーブルリストのティア名をクリックして既存のティアを編集します。便宜上、新しく作成された階層は最初に __default__ ティアの設定を継承します。ティアを活用して SaaS アプリケーションデリバリーを最適化する方法の詳細については、ユーザーガイドを参照してください。

ティアを定義したら、次はアプリケーションとそのサービスの設定を行う必要があります。画面左側のナビゲーションから **Application** を選択します。次のようなページが表示されます。

![Application Setup](../images/gs-app-setup.png?raw=true "Application Setup")

まず、アプリケーションにわかりやすい **Name** を付けてください。テスト用に **Domain Name** または **SSL Certificate** エントリを入力する必要はありません。本番環境では、SaaS アプリケーションをホストするドメイン名の証明書が [AWS Certificate Manager](https://aws.amazon.com/certificate-manager/) で定義されていることを確認してください。AWS SaaS Boost を設定する前に、[ドメイン名の登録](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/domain-register.html) と [SSL 証明書の設定](https://docs.aws.amazon.com/acm/latest/userguide/gs.html) を実行する必要があることに注意してください。

AWS SaaS Boost では、ワークロードをサポートするのに必要な数の「サービス」を設定できます。サービスごとに個別の Docker イメージを提供します。これらのサービスはパブリックでもプライベートでもよく、互いに独立して構成されます。サービスはファイルシステムやデータベースのようなリソースを共有しませんが、プロビジョニングされた VPC ネットワーク内で相互に通信できます。パブリックにアクセス可能なサービスは、Application Load Balancer を介して公開され、インターネット経由で DNS 経由でアクセスできます。プライベートサービスにはインターネットからアクセスできません。SaaS アプリケーションに最適なサービスの使用方法について詳しくは、開発者ガイドとユーザーガイドを参照してください。

**New Service** ボタンをクリックして、最初のサービスを作成します。次のようなポップアップダイアログが表示されます。


![Application Setup](../images/gs-new-service.png?raw=true "New Service")

この例ではサービスを `main` と呼んでいます。ポップアップダイアログの **Create Service** ボタンをクリックすると、次のような新しいサービスがサービスのリストに表示されます。

![Application Setup](../images/gs-service-collapsed.png?raw=true "Service Config Collapsed")

サービス名をクリックすると、このサービスの設定オプションが展開されます。SaaS アプリケーションを構成する各サービスの設定とティアベースのバリエーションを設定するための動的フォームが表示されます。

この_入門ガイド_は設定画面のすべてのフィールドを説明しているわけではありませんが、選択するオプションはアプリケーションを機能させるために不可欠です。SaaS Boostで提供されるサンプルアプリケーションの1つに記入されたサービス設定フォームの例については、以下を参照してください。

## ステップ 6 - アプリケーションサービスの更新
定義した各ティアに各サービスを設定すると、SaaS Boost は各サービスの [Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html) イメージリポジトリを自動的に作成します。テナントをシステムにオンボーディングする前に、アプリケーション内の各サービスの Docker イメージをアップロードする必要があります。**Summary** ページから、各サービスの `ECR Repository URLL` リストの横にある **View detail示** リンクをクリックすると、イメージをアップロードするための適切な Docker プッシュコマンドが表示されます。サンプルアプリに含まれているビルドシェルスクリプトを参照して、Docker プッシュプロセスを自動化する 1 つの方法を確認することもできます。

## ステップ 7 - (オプション) サンプルアプリケーショのデプロイ
このセクションでは、サンプルアプリケーションをアップロードするプロセスについて説明し、このプロセスがどのように機能するかを理解してもらいます。AWS SaaS Boost リポジトリの一部として、簡単なサンプルアプリケーションを提供しています。

### サンプルアプリケーション用の AWS SaaS Boost の設定
アプリケーションの要件をサポートするように AWS SaaS Boost を設定するのと同様に、このサンプルアプリケーションのアプリケーション設定を適切に設定する必要があります。このサンプルアプリケーションは、AWS SaaS Boost の以下の設定に依存しています。
- **Sample** など、アプリケーションのわかりやすい名前 `Name` を入力
- `New Service`」を作成し、それに **main** のような名前を設定
- `Publicly accessible` ボックスが **on** になっていることを確認
- `Service Addressible Path` が **/\*** になっていることを確認
- `Compute` セクション:
      - `Container OS` に **Linux** を選択
      - `Container Tag` を **latest** に設定
      - `Container Port` を **8080** に設定
      - `Health Check URL` を **/index.html** に設定
- `Filesystem` セクション:
      - `Provision a File System for the application` チェックボックスを **Enable** にし、**EFS** を選択
      - `Mount point` を **/mnt** に設定
- `Database` セクション:
      - `Provision a database for the application` チェックボックスを **Enable** に設定
      - 使用可能なデータベースのいずれかを選択します（db.t3.micro インスタンスクラスの MariaDB がおそらく最も速くプロビジョニングされます）
      -　**Database Name**、**Username**、**Password** を入力。データベースを初期化するためのSQLファイルの提供は不要。
- `default` ティア設定以下:
      - `Compute Size` を **Medium** に設定
      - `Minimum Instance Count` と `Maximum Instance Count` をそれぞれ **1** と **1** に設定
      - データベース用に `db.t3.micro` インスタンスクラスを設定

設定は以下のようになるはずです。

![Sample App Config](../images/gs-sample-app-config.png?raw=true "Sample App Config")

### サンプルアプリケーションのビルドとデプロイ
アプリケーション設定を保存すると、SaaS Boost はアプリケーション内の定義済みサービスごとに ECR リポジトリを作成します。CloudFormation  がSaaS Boost インストールの更新を完了したら、サンプルアプリケーションをビルドしてコンテナ化できます。このサンプルアプリケーションの Docker イメージを作成するには、ローカルマシンで Docker を実行している必要があります。AWS SaaS Boost リポジトリのクローンにある **samples/java/** ディレクトリに移動し、ビルドスクリプトを実行します。これにより、コンテナ化されたアプリケーションがビルド、コンテナ化され、AWS SaaS Boost ECR リポジトリにプッシュされます。このシェルスクリプト例の手順を確認して、アプリケーションの現在のビルドプロセスを強化して AWS SaaS Boost と統合する方法を確認できます。

```shell
cd aws-saas-boost/samples/java
sh build.sh
```

このスクリプトは、AWS SaaS Boost 環境の入力を求めます。AWS SaaS Boost インストーラを実行したときに指定した環境ラベルを入力します。次に、スクリプトは、構築するサービスの名前の入力を求めます。この例では `main` を使用します。サービス名は大文字と小文字が区別されることに注意してください。このスクリプトが完了すると、アプリケーションサービスが AWS SaaS Boost アプリケーションリポジトリに公開されます。アプリケーションに変更を加えると、ビルドスクリプトを再度実行してアプリケーションを更新できます。

マルチサービスアプリケーションでは、設定するサービスごとに同じ手順を繰り返します。

### テナンtののオンボーディング
これで、アプリケーションをサポートするために必要なすべてのインフラストラクチャをプロビジョニングし、設定したすべてのサービスをデプロイする新しいテナントをオンボーディングできます。管理アプリケーションの **Onboarding** ページに移動し、**Provision Tenant** ボタンをクリックします。オンボーディングプロセスが完了すると、管理アプリケーションの **Tenant** ページに移動し、テナント詳細ページに移動し、最後に **Load Balancer DNS** リンクをクリックすることで、サンプルアプリケーションのそのテナントのインスタンスにアクセスできるようになります。

## アプリケーションと AWS SaaS Boost のマッピン
このガイドでは、AWS SaaS Boost の設定に必要な基本ステップの概要を説明します。ワークロードを AWS SaaS Boost に移行することを検討する際には、AWS SaaS Boost の幅広い機能をさらに詳しく調べてください。つまり、アプリケーションの設定オプションと、それらがさまざまな AWS SaaS Boost アプリケーション設定にどのようにマッピングされるかをより注意深く検討する必要があります。

全体的なエクスペリエンスに慣れれば、アプリケーションが AWS SaaS Boost モデルにどのように適合するかをよりよく理解できるようになります。AWS SaaS Boost [ユーザーガイド](user-guide.ja.md) と [開発者ガイド](developer-guide.ja.md) を確認しておくと、ソリューションのニーズに合わせて AWS SaaS Boost を設定する方法をよりよく理解できるようになります。

The steps needed to containerize your workload vary depending on the nature of your application. You can use the Dockerfile examples in the sample apps provided with AWS SaaS Boost. Here are some additional resources that provide information on containerizing applications:
- [AWS の学習過程: モノリスのコンテナ化](https://aws.amazon.com/jp/getting-started/container-microservices-tutorial/module-one/)
- [AWS App2Container](https://aws.amazon.com/jp/app2container/)
