# AWS SaaS Boost Developer Guide

## Contents
[イントロダクション](#イントロダクション)\
[ハイレベルアーキテクチャ](#ハイレベルアーキテクチャ)\
[アドミンアプリケーション](#アドミンアプリケーション)\
[コアサービス](#コアサービス)\
[オンボーディングサービス](#オンボーディングサービス)\
[テナントサービス](#テナントサービス)\
[ユーザーサービス](#ユーザーサービス)\
[設定サービス](#設定サービス)\
[メトリクスサービス](#メトリクスサービス)\
[クオータサービス](#クオータサービス)\
[ビリング統合](#ビリング統合)\
[メトリクスと分析](#メトリクスと分析)\
[プリケーション更新のデプロイ](#プリケーション更新のデプロイ)

## イントロダクション
このドキュメントでは、software as a service (SaaS) の開発者とアーキテクトに SaaS Boost テクノロジーの内部像を説明します。SaaS Boost の体験全体を定義するサービスとテクノロジーについて説明します。

オープンソースプロダクトとして、SaaS Boost 環境のアーキテクチャと設計は継続的に進化します。したがって、このドキュメントの目的は、各 API 呼び出しのシグネチャとコントラクトではなく、SaaS Boost アーキテクチャのコアビルディングブロックに重点を置くことです。SaaS Boost アーキテクチャの全体的な技術的展望を概説し、開発者がシステムのハイレベルのアーキテクチャ要素を理解するのに役立つ洞察を提供します。

## ハイレベルアーキテクチャ
SaaS Boost アーキテクチャには 4 つの主要なレイヤーがあります。これらの各レイヤーは、システムの全体的なアーキテクチャにおいて異なる役割を果たします。図 1 は、これらのコアコンセプトの概要を示しています。

![High-Level Architecture](../images/dg-high-level-arch.png?raw=true "High-Level Architecture")
<p align="center">Figure 1 - High-level architecture</p>

図 1 の左側は、SaaS Boost 環境の基本コンポーネントを示しています。管理アプリケーションは環境への管理ユーザーインターフェイスを提供し、SaaS Boost 環境の属性を設定および管理できるようにします。このアプリケーションは、SaaS ソリューションのプロビジョニング、管理、運用に使用されるすべてのマイクロサービスを表すコアサービスレイヤーと相互作用します。環境のコードの大部分はここにあります。また、カスタマイズが導入される可能性が高い場所でもあります。

下のレイヤーは、より多くのオペレーション構成（ビリング、メータリング、メトリクス、分析サービス）を表しています。これらのサービスは堅牢な SaaS 環境に不可欠ですが、コアサービスと各テナントの環境で実行されるアプリケーションの両方で使用されるため、図では別々に表示されています。また、これらのサービスでは、開発者は、オペレーションデータやメトリクスをこれらのサービスに公開できる追加のコードをアプリケーションに組み込む必要があります。

このモデルの最後のレイヤーは、管理対象テナント環境です。このレイヤーは、システムの各テナントにプロビジョニングされるリソースを表します。実際には、このレイヤーに対応するコードはリポジトリにありません。代わりに、コアサービスには、マネージドテナント環境レイヤーで実行されるすべてのアーキテクチャをプロビジョニングして構成するコードがあります。それでも、テナントを使用してシステムを稼働させると、SaaS Boost 環境によって管理される各テナントのアーキテクチャ構成がプロビジョニングされます。

## アドミンアプリケーション
SaaS Boost [ユーザーガイド](user-guide.ja.md) では、管理アプリケーションの機能について詳しく説明しています。ここでは、このアプリケーションの構築に使用されたテクノロジーとアーキテクチャにさらに焦点を当てます。

管理アプリケーションは、React JS フレームワークを使用してシングルページアプリケーション (SPA) として構築されます。これはあらゆる React アプリケーションの構築に使われる典型的なパターンや慣習に準拠しています。また、ブートストラップツールキットを実行する CoreUI 管理テンプレートも使用します。

![Administration Application Architecture](images/dg-admin-app-arch.png?raw=true "Administration Application Architecture")
<p align="center">Figure 2 - Administration application architecture</p>

図 2 は、クライアント環境のアーキテクチャフットプリントを示しています。示されているサービスは、SPA をホストするために使用される一般的な AWS アーキテクチャを表しています。

このソリューションでは、Amazon CloudFront を管理アプリケーションのエントリポイントとして使用します。このディストリビューションは、受信リクエストを SaaS Boost React アプリケーションをホストする Amazon S3 バケットに転送します。このアプリケーションへのアクセスは、SaaS Boost 管理エクスペリエンスへのアクセスを許可されたすべてのユーザーを管理する Amazon Cognito によって制御されます。

### クライアントのローカル実行
SaaS Boost をインストールして実行したら、管理アプリケーションに変更を加えることに興味があるかもしれません。変更を加えてローカルで開発/テストしたい場合は、SaaS Boost リポジトリのクローン時にダウンロードしたコードの一部である `/client/web` フォルダに移動できます。

同じフォルダ内には、デプロイされた環境に固有の変数を格納する `sample.env` ファイルがあります。このファイルの名前を `.env` に変更し、ローカル環境がデプロイされた SaaS Boost 環境のコアサービスに接続できるように設定を編集します。

新しく名前を付けたファイルを、任意のエディターで開きます。クライアントをローカルで実行するには、さまざまな環境変数を構成する必要があります。以下は、構成された設定の例です。
```ini
AWS_DEFAULT_REGION=us-east-1
REACT_APP_AWS_REGION=us-east-1
REACT_APP_COGNITO_USERPOOL=us-east-1_XXXXXXXXX
REACT_APP_CLIENT_ID=[YOUR CLIENT ID HERE]
REACT_APP_API_URI=[API URI HERE]
```
これらの設定の値は、AWS マネジメントコンソールから取得できます。リージョンは SaaS Boost デプロイメントの物理的な場所に基づいています。コンソールで Amazon Cognito サービスにアクセスして、ユーザープールとクライアント ID の設定を調べることもできます。バックエンドサービスの URI エントリポイントを取得するには、コンソールで API Gateway サービスを使用し、パブリック API の v1 ステージの呼び出し URL をコピーします。

これらの環境変数を正しく設定したら、次のコマンドを使用して `/client/web` フォルダからクライアントをローカルで編集して実行できます。`yarn start`

## コアサービス
SaaS Boostのコアサービスは、環境によってサポートされる面倒な作業の多くを担っています。SaaS 環境のプロビジョニング、管理、運用に必要なマイクロサービスと接続に注目してください。

詳細を掘り下げる前に、図 3 に示すように、コアサービスの概要を見てみましょう。

![Core Services](../images/dg-core-services.png?raw=true "Core Services")
<p align="center">Figure 3 - Core services</p>

コアサービスのアーキテクチャは、AWS Lambda のサーバーレスモデルで実行される一連のマイクロサービスとして実装されます。これらのサービスに課せられる負荷は非常に変動しやすいため、通常、サーバーレスモデルのコストと消費プロファイルによく合います。これにより、環境内でアイドル状態のサービスや使用頻度の低いサービスにお金を払う必要がなくなります。

管理アプリケーションと各 SaaS テナント環境は、SaaS Boost のパブリック API を公開する Amazon API ゲートウェイを介してこれらのサービスにアクセスします。

各サービスは、機能を提供するためにさまざまな AWS サービスに依存しています。サービスは一般的なマイクロサービスのベストプラクティスに準拠しており、その実装に使用されるストレージと基盤となる構成をカプセル化しています。各サービスは、そのコントラクトを表す API をサポートします。

ここで紹介するサービスは、SaaS Boostのベースライン機能であり、現在システムでサポートされている特定のユースケースに対応しています。SaaS Boostが新しいSaaSモデルとユースケースのサポートを追加するにつれて、サービスリストは増えるでしょう。

以下のセクションでは、システムのマイクロサービスの実装について説明し、各サービスの実装の目標と範囲について説明します。これにより、開発者はこれらのサービスの設計に慣れることができ、ユーザーはどこでどのように環境をカスタマイズし、アプリケーションのニーズに合わせればよいかがわかります。

## オンボーディングサービス
オンボーディングサービスは、SaaS Boost 体験の重要なコンポーネントの1つです。SaaS 環境の作成、設定、更新をオーケストレートします。これには、新しいテナントのオンボーディングとアプリケーション更新のデプロイの両方が含まれます。

これらすべてを SaaS Boost 内で管理および展開することで、オンボーディングプロセスの一部となるすべての戦略とポリシーを環境でサポートできるようになります。つまり、テナント環境のアーキテクチャ、構成、およびデプロイポリシーは、オンボーディング体験に影響を与えることなく、時間の経過とともに変化する可能性があります。多くの点で、オンボーディングはシステムの新たなニーズに応じて進化するブラックボックスであることを望んでいます。

オンボーディングサービスには、新しいテナントのオンボーディングに必要なデータを受け入れる API が含まれています。これにより、2 つのオンボーディングパスが開きます。システムがセルフサービスのオンボーディング体験をサポートしている場合は、アプリケーションにオンボーディングページを追加してオンボーディングをトリガーできます。システムがセルフサービスオンボーディングをサポートしていない場合は、SaaS Boost 管理アプリケーションからオンボーディングできます。図 4 は、オンボーディングプロセスの概念的な概要を示しています。

図 4 の左側の画像は、新しいテナントをオンボーディングするための 2 つの異なる方法を示しており、オンボーディングリクエストをセルフサービスまたは管理アプリケーションに送信する方法を示しています。リクエストは、新しいテナントに関するすべてのデータを保持する JSON メッセージとしてオンボーディングサービスの API に送信されます。

![Onboarding Flow](../images/dg-onboarding-flow.png?raw=true "Onboarding Flow")
<p align="center">Figure 4 - The onboarding flow</p>

新しいテナントをオンボーディングするために、オンボーディングサービスは構成データを使用して次のリソースを作成および構成します。
- **テナント環境のプロビジョニング**: このプロセスにより、各テナントの仮想プライベートネットワーク (VPC)、ネットワーク構成、Amazon ECS クラスター、Route 53 ドメインネームシステム (DNS) エントリ、およびその他のインフラストラクチャが作成されます。
- **プロビジョニング拡張**: 拡張機能は、特定の SaaS アプリケーションに固有のインフラストラクチャのオプション部分です。これらの拡張機能 (データベース、ファイルシステムなど) は、アプリケーション設定の一部として構成され、オンボーディングプロセスの一環としてデプロイ/設定されます。
- **テナントのプロビジョニング**: オンボーディングプロセス中に送信されたデータを使用して、システム内にテナントの署名を作成します。
- **ビリングアカウントの作成**: SaaS Boost の組み込みビリング機能を使用する場合、アプリケーションはそのビリングシステムで新しい顧客を作成する必要があります。オンボーディングプロセスでは、ビリング統合の一環として新規顧客を作成します。
- **アプリケーションのデプロイ**: テナントのインフラストラクチャが整ったら、そのインフラストラクチャに SaaS アプリケーションをデプロイする必要があります。オンボーディングでは、SaaS Boost にアップロードされたアプリケーションがデプロイされます。

これにより、SaaS Boost モデル全体におけるオンボーディングサービスの重要性が理解できるはずです。オンボーディングは、特にサイロ化されたSaaSモデルにおいて、SaaS環境の構築に不可欠な自動化の多くの中心です。

また、このプロセスの背後にあるコードが、既存のモデルを拡張する可能性が高い領域であることも明確にしておく必要があります。たとえば、新しい拡張機能を追加すると、デフォルトの SaaS Boost エクスペリエンスではサポートされていないカスタムコンセプトを導入するのが自然かもしれません。

次のセクションでは、オンボーディングプロセスの詳細を説明します。

### テナントインフラストラクチャをプロビジョニング
テナントインフラストラクチャのプロビジョニングとは、各テナントの環境をホストするために必要なインフラストラクチャを作成することです。サイロモデルで運用しているため、ここでプロビジョニングされるインフラストラクチャの量はより顕著です。プールモデル（共有インフラストラクチャ）を使用すれば、オンボーディングエクスペリエンスに変動する部分は少なくなります。図 5 は、これらのテナント環境の主要な要素を示しています。

![Provisioned Tenant Environments](../images/dg-tenant-environments.png?raw=true "Provisioned Tenant Environments")
<p align="center">Figure 5 - Provisioned tenant environments</p>

各テナントは個別のマルチ AZ VPC にデプロイされることに注意してください。これらの VPC は Fargate ベースの Amazon ECS クラスターを実行し、VPC エントリポイントごとに個別のアプリケーションロードバランサーを備えています。Amazon Route 53 は、各テナントサブドメイン (例:`tenant1.abc.com`) をテナントのインフラストラクチャをホストする対応する VPC にルーティングするように設定されています。

一部の構成オプションは、インフラストラクチャ内のテナント体験のパフォーマンスを変更するために使用されます。たとえば、計算クラスターのスケーリングプロファイルは、特定のテナントのティアによって異なる場合があります。

環境をプロビジョニングするコードは、主に AWS CloudFormation によって駆動されます。オンボーディングサービスは、AWS CloudFormation スタックを起動するための呼び出しを呼び出し、モニタリングします。多くの場合、テナント環境の拡張とカスタマイズは、基盤とな るCloudFormation テンプレートを変更することで実現できます。

### プロビジョニング拡張
完全な SaaS 環境の構築に伴う主な課題は、各 SaaS アプリケーションがコンピューティング以外にもインフラストラクチャに依存している可能性があることです。たとえば、ほとんどのアプリケーションはデータベースに依存していますが、どのデータベースがどのように構成されるかはアプリケーションごとに異なります。

これらのバリエーションをサポートするために、SaaS Boost はアプリケーションで必要となる可能性のある追加リソースを設定できる拡張機能を使用します。SaaS Boost はモノリシック環境から SaaS への移行に重点を置いているため、選択した拡張機能はモノリシック環境でより一般的なサービスに偏っています。
![Provisioning Extensions](../images/dg-extensions.png?raw=true "Provisioning Extensions")
<p align="center">Figure 6 - Provisioning extensions</p>

図 6 は、拡張モデルの概要を示しています。各拡張には、設定オプションを格納するための、対応する AWS CloudFormation ファイルがあります。各テナントがプロビジョニングされると、パラメータが AWS CloudFormation スタックに渡され、各テナントの設定が伝達されます。

サポートされている拡張機能のタイプは、オペレーティングシステムによって決まります。Linux 環境と Windows 環境には、有効にできるオプションに影響する特定の要件があります。この段階では、主にデータベースとファイルシステムに重点が置かれ、最終的にはさらに多くの拡張機能がサポートされることが予想されます。

独自の拡張機能を導入したい場合は、AWS CloudFormation スタックを実行するコードに慣れておいてください。拡張機能は、開発者が SaaS 環境のニーズをサポートする新しい構成オプションを導入する自然な機会です。

### ビリングアカウントの作成 (有効化されている場合)
新しいテナントがシステムに登録されるたびに、SaaS Boost と統合されたビリングシステム内に関連テナントが作成されます。SaaS Boostは、オンボーディングプロセス中にこのテナント情報をキャプチャし、SaaS アプリケーションがビリングイベントを発行するときに参照します。

オンボーディングプロセスでは、オンボーディングイベントをビリングシステムに発行することでテナントを作成します。このイベントには、ビリングシステムで新規テナントを作成するために必要なすべてのデータが含まれています。このメッセージは Amazon EventBridge に発行されます。Amazon EventBridge は受信リクエストを受け取り、サードパーティのビリング API を呼び出して新しい顧客を作成します。

### アプリケーションデプロイ
オンボーディングフローの最後の部分は、新しいテナント環境にアプリケーションをデプロイすることです。このデプロイは、Amazon ECR にアップロードしたコンテナ化されたイメージを使用し、テナント環境のプロビジョニング中に作成された Amazon ECS クラスターにデプロイします。

![Deploy Application for Tenant](../images/dg-deploy.png?raw=true "Deploy Application for Tenant")
<p align="center">Figure 7 - Deploy application for new tenant</p>

図 7 は、デプロイプロセスの例を示しています。テナントオンボーディングマイクロサービスは、AWS CodePipeline を使用してアプリケーションのデプロイをオーケストレートします。ECR からコンテナイメージを取得し、ECS クラスターへのローリングデプロイを開始します。このステップが完了すると、テナントは新しい環境にアクセスして SaaS アプリケーションを実行できます。

### オンボーディングステータスの確認と管理
オンボーディングマイクロサービスは、オンボーディングフローの各ステップのステータスを追跡します。ステップはキャプチャされて表示されます。これにより、オンボーディングライフサイクルの進捗と状態を可視化できます。状態は管理アプリケーションに表示されます。

これらの状態は情報提供のみを目的としていますが、ダウンストリームのイベントをトリガーするためにフローの段階で監視されることもあります。そのため、多くの場合、システムは特定の状態遷移を待ってから処理を進めることができます。

![Onboarding States](../images/dg-onboarding-status.png?raw=true "Onboarding States")
<p align="center">Figure 8 - Onboarding states</p>

図 8 は、オンボーディングフローのさまざまな状態を示しています。まず、テナントマイクロサービスを使用してテナントを作成します。テナントが作成されると、オンボーディングフローは「Created」状態に移行します。次に、オンボーディングフローでは AWS CloudFormation スタックを使用して、必要な拡張機能を含む各テナントに必要なインフラストラクチャを作成します。このプロセスが起動すると、オンボーディングフローは「Provisioning」状態に移行します。

ワークフローはプロビジョニング状態を追跡するのにAmazon Simple Notification Service (Amazon SNS) リスナーに依存していることに注意してください。基本的に、このリスナーはテナントインフラストラクチャのプロビジョニングが完了するのを待つことで、AWS CloudFormation の進行状況を追跡します。このプロセスが完了すると、オンボーディングワークフローは「Provisioned」状態に移行します。

この段階では、作成したテナントリソースに関するデータがあり、後で参照できるように保持する必要があります。プロビジョニングプロセス中に作成されたリソースに関するデータをキャプチャして保存するための AWS Lambda 関数があります。また、この関数はテナントオンボーディングイベントを Amazon EventBridge に送信して、ビリングシステムに新しいテナントアカウントをプロビジョニングします。Lambda 関数の処理が終了すると、システムは「Deploying」状態に移行します。

このプロセスの最後のステップは、アプリケーションをテナントの新しい ECS クラスターにデプロイする AWS CodePipeline によって管理されます。Amazon EventBridge は AWS CodePipeline の進行状況を追跡し、パイプラインが終了したと判断するとワークフローを「デプロイ済み」状態に移行します。


## テナントサービス
Every SaaS system must have a centralized mechanism for tracking and managing the state of individual tenants. This is the role of the tenant microservice, which provides a collection of basic operations that can be performed on any tenant. This includes creating new tenants (as part of onboarding), updating tenant settings, and enabling/disabling tenants.

The tenant service is implemented as a collection of Lambda functions, each of which supports a different operation. The data for this service is stored in Amazon DynamoDB, where each item is accessed by a tenant identifier that's stored in a partition key of a DynamoDB table. This data includes a range of attributes that represent information about a tenant's configuration and status (enabled/disabled).

When onboarding a tenant, you have the option to specify settings for that tenant's environment. This allows you to size the tenant based on their given tier. This means the compute size, for example, could be configured differently for basic- and advanced-tier tenants.

![Tenant Configuration](../images/dg-tenant-configuration.png?raw=true "Tenant Configuration")
<p align="center">Figure 9 - Tenant configuration</p>

図 9 は、テナント構成の例を示しています。これは、SaaS Boost 管理アプリケーションでサポートされているビルトインテナントオンボーディングを表しています。「Override Application Defaults」チェックボックスに注意してください。オンにすると、アプリケーションレベルで構成されているコンピューティング設定を上書きできます。

これらの設定オプションは各テナントに保存され、テナントのオンボーディング時にプロビジョニングの一部として使用されます。値を個別に更新してテナントに適用することもできます。

テナントレベルで個別のサイジング属性を提供する機能は、ティアリング戦略の一部として使用できます。ただし、テナントごとに個別のスケーリング設定を行うことは避けることをお勧めします。これにより、SaaS システムの一部として目標としているオペレーション効率が損なわれる可能性があります。システムの各ティアに標準のスケーリング値セットを定義し、テナントをプロビジョニングする際にこれらのティア値を使用することをおすすめします。アプリケーションレベルで設定する設定は、デフォルティア層を表します。ティアは最終的に、SaaS Boost 内のサイジングとスケーリングを管理する正式な構成要素になるでしょう。

テナントサービスは、テナントのステータスを選択的に変更できる有効化/無効化オプションもサポートしています。テナントを無効にするようにサービスに要求すると、そのテナントのアプリケーションへのすべてのアクセスは、再び有効になるまでブロックされます。

## ユーザーサービス
ユーザーマイクロサービスは、管理アプリケーションのユーザーを管理するための操作を提供します。管理者のリストを管理するための従来の作成、読み取り、更新、削除 (CRUD) メカニズムをサポートしています。Amazon Cognito は各ユーザーのデータを管理します。つまり、各管理オペレーションは Amazon Cognito に API 呼び出しを行い、ユーザーを取得、作成、更新、または削除します。

## 設定サービス
設定サービスは、さまざまな SaaS Boost サービスで使用されるさまざまな設定を一元管理するメカニズムを提供します。多くの点で、このサービスは基本的なプロパティ/バリューストアとして実装されています。つまり、グローバル構成データに加えて、インフラストラクチャプロビジョニングからの主要なアウトプットを収集するのに役立つ情報が格納されています。

![Settings Service](../images/dg-settings-service.png?raw=true "Settings Service")
<p align="center">Figure 10 - Settings service</p>

図 10 は、設定サービスの概要を示しています。SaaS Boost 設定には主に2つのプロデューサーがあります。図の右側では、SaaS Boostの一部であるインストールスクリプトが、設定サービスによって保存されるさまざまなインフラストラクチャ環境パラメーターを公開しています。保存されるプロパティと値は、SaaS Boost 環境のプロビジョニング時に作成されるさまざまなインフラストラクチャリソースに対応しています。これにより、ダウンストリームのプロセスとスクリプトがインフラストラクチャリソースへの参照を解決できます。

図の左側では、管理アプリケーションは、アプリケーションの構成に使用されるすべてのデフォルト設定を送信することにより、構成データを設定サービスに公開します。これには、コンピューティング、オペレーティングシステム、拡張機能 (データベース、ファイルシステムなど)、ビリング統合情報に加えて、アプリケーションドメインの設定が含まれます。

設定サービスはオンボーディングプロセスでも使用されます。オンボーディングプロセスでは、これらの設定に基づいて、システムに新しいテナントが追加されるたびに構成されます。設定サービスは、AWS Systems Manager パラメータストアを使用して設定を保存します。

## メトリクスサービス
SaaS Boost には、組織がテナント環境のアクティビティと状態を継続的に評価できる体験が組み込まれています。このサービスのために収集されるデータは、管理アプリケーションがオペレーション分析のチャートやグラフにアクセスして表示するための手段を提供するメトリックサービスから取得されます。

現在、メトリクスサービスは、各テナントのコンピューティングリソースを実行する Amazon ECS クラスターからのデータアクティビティを監視しています。また、Application Load Balancer のアクセスログからアクセスパターンに関するデータを収集します。図 11 は、メトリクス・マイクロサービスの主要部分を示しています。

![Inside the Metrics Service](..images/dg-metrics-service.png?raw=true "Inside the Metrics Service")
<p align="center">Figure 11 - Inside the metrics service</p>

図 11 は、各テナント環境の一部であるインフラストラクチャを示しています。Amazon ECS の消費メトリクスは Amazon CloudWatch を通じて公開され、アクセスパターンデータはキャプチャされて S3 バケットに保存されます。管理アプリケーションがメトリクスマイクロサービスにアクセスしてグラフをレンダリングすると、これらの呼び出しにより、ここで説明する 2 つのソースからグラフデータセットが構築されます。

メトリクスサービスは CloudWatch から消費データを表示するグラフのデータを取得し、アクセストレンドのグラフは Amazon Athena を呼び出してアクセスログをクエリします。次に、データは JSON 構造に変換され、グラフのデータポイントの標準化された表現が返されます。

ここで重要なのは、メトリクスサービスの焦点は、テナントを意識した特定のオペレーション洞察へのアクセスを可能にすることにあるということです。テナントコンテキストは、ここでキャプチャされるすべてのメトリックデータの一部です。このコンテキストを使用してデータのビューを作成し、SaaS プロバイダーがテナント (または階層) の観点からシステムアクティビティを分析および評価できるようにします。管理アプリケーションでは、可能な場合はテナント別に分析ビューをフィルタリングできます。これにより、個々のテナントの傾向に焦点を当てることができ、運用チームを支援できます。

## クオータサービス
クオータサービスは、AWS アカウントクォータに関する情報を収集するための軽量なメカニズムを提供します。新しいテナントがシステムに追加されると、さまざまなアカウント制限の状態を評価して、システムが新しいインフラストラクチャをプロビジョニングする準備ができているかどうかを判断します。

このマイクロサービスは、さまざまな AWS サービスにさまざまな呼び出しを行い、クオータデータを収集します。これらの呼び出しから返されたデータは、SaaS Boost 環境が、より多くのテナントをプロビジョニングするシステムの能力に影響を与える可能性のあるクオータの閾値を超えているかどうかを評価するために使用されます。

## ビリング統合
SaaS Boost は、企業にビリングシステムとの統合オプションを提供します。これにより、SaaS モデルに移行するユーザーは、すぐに使える課金ソリューションを利用できるようになり、SaaS プロバイダーは最小限の労力でビリングサポートを導入できるようになります。

ビリング統合を提供する場合の課題は、SaaS ビリングシステム用の標準 API やモデルがないことです。共通のテーマはありますが、各プロバイダーにはそれぞれのビリング体験を表現する独自のメカニズムがあります。これを軽減するために、サードパーティの請求システム (Stripe) と SaaS Boost の間にレイヤーを導入しました。

ビリング統合では、SaaS Boost とアプリケーションに導入した変更の両方を使用して完全なビリング体験を構築することに注意してください。SaaS Boost は、可能な限り多くの可動部分を構成します。SaaS Boost でビリングを有効にしたら、SaaS 請求書の生成に使用されるイベントを公開するのはアプリケーションの責任です。アプリケーションからのこの統合は、ユーザーとビリングシステムの間にあるビリング統合レイヤーに依存しています。

Stripe のサービスの使用には、AWS との契約は適用されません。お客様による Stripe のサービスの利用には、個人データの処理に関する Stripe の規約を含む Stripe の法的条件が適用されます。[こちら](https://stripe.com/legal)をご覧ください。AWS サービスの使用はすべて、AWS SaaS Boost の使用にかかる料金に加えて、該当する AWS アカウントに請求されることに注意してください。AWS コンソールで、各 AWS アカウントの AWS サービスの使用状況を確認できます。

![Billing Integration](../images/dg-billing-integration.png?raw=true "Billing Integration")
<p align="center">Figure 12 - Billing integration</p>

図 12 は、ビリング統合の中核となる可動部分を示しています。ビリングアーキテクチャには 4 つの異なる要素があり、それぞれ異なる色分けスキームで表現されています。図の右側にはステップ 1 と 2 があり、SaaS プロバイダーは Stripe アカウントを作成することから始めます。この時点で、SaaS Boost 統合に必要な API キーを取得できます。

Stripe アカウントを作成したら、SaaS Boost 管理アプリケーションから API キーを入力します (ステップ 3 ～ 5)。システムは設定マイクロサービスを使用してこの API キーを保存します。これにより、システムへのビリングが可能になります。この API を設定すると、アプリケーションの請求モデルの決定に使用される Stripe 製品も設定されます。

これで、新しいテナントのオンボーディングをサポートするための請求項目が整いました。ステップ 6 ～ 9 はオンボーディングフローです。アプリケーションまたは管理アプリケーションがテナントをオンボーディングすると、アプリケーションはオンボーディングサービスにメッセージを送信します。その後、システムは (Amazon EventBridge 経由で) ビリング統合にメッセージを送信します。これにより、Stripe API への呼び出しがトリガーされ、Stripe 内で新規顧客のアカウントが作成されます。ストライプは Amazon DynamoDB に保存されているサブスクリプションに関するデータを返します。

オンボーディングフローの最後の部分では、アプリケーションからのビリングイベントの発行に焦点を当てます（ステップ 10 ～ 12）。ここで、アプリケーション内の主要な請求アクティビティを特定し、Stripe に送信して請求額を計算します。これらのイベントを処理するには、システムは Stripe にメッセージを送信する前に、テナントのサブスクリプション情報を調べる必要があります (ステップ 11)。

## メトリクスと分析
SaaS Boost にはメトリクスと分析ツールが組み込まれていますが、SaaS プロバイダーにとっては、ドメイン/環境に固有のメトリックをキャプチャして分析するためのより広範なメカニズムを持つことが不可欠です。メトリクスは、さまざまなユースケースやコンシューマーに及ぶ可能性があります。

これらのカスタムメトリックの収集は、ビジネスに不可欠なすべてのメトリックデータを SaaS アプリケーションに組み込むことで実現されます。このデータから抽出する価値は、多くの場合、アプリケーションが公開するデータの質と深さに直接関係しています。

これをサポートするために、組織がメトリクスデータを発行、集計、分析できるようにする従来のデータ取り込みアーキテクチャを作成しています。図 13 は、SaaS Boost メトリクス・インフラストラクチャーの概要を示しています。このモデルは、データ分析の課題に対処するために一般的に使用される一般的なデータ取り込みと集約のメカニズムに従っています。

![Metrics and Analytics](../images/dg-metrics-analytics.png?raw=true "Metrics and Analytics")
<p align="center">Figure 13 - Metrics ingesting, aggregation, and visualization</p>

図 13 はテナント環境を示しています。この環境では、関連するメトリクスを発行するコードを追加できます。このデータは Amazon Kinesis Firehose によって取り込まれ、Amazon Redshift に移動されます。この時点で、Amazon QuickSight を使用して、SaaS 環境のさまざまなオペレーション、アーキテクチャ、ビジネス、テナントの傾向を分析するダッシュボードを構築できます。

アプリケーションから公開されたデータは、消費情報を伝える汎用 JSON イベントにパッケージ化されます。このイベントには JSON モデルが提供されていますが、特定のアプリケーションや環境のニーズに基づいてイベントの構造を調整することもできます。

メトリクスアーキテクチャは SaaS Boost のオプションコンポーネントであることに注意してください。これは自分の環境にとって意味のあるときに導入できます。これらのメトリクスを持つことは SaaS ビジネスにとって不可欠ですが、この要件に対処するために他のツールを使用することもあることを認識しています。

## アプリケーション更新のデプロイ
環境が稼働し、いくつかのテナントをオンボーディングしたら、チームが次に重点的に取り組むのはアプリケーション更新のデプロイです。これらの更新のデプロイは、新しいリリースが毎回システムのすべてのテナントにデプロイされる SaaS 環境では特に重要です。SaaS Boost はこのデプロイライフサイクル全体を処理し、新しいバージョンをローリングアップデートとしてすべてのテナント環境に公開します。

![Deploying Application Updates](../images/dg-deploying-updates.png?raw=true "Deploying Application Updates")
<p align="center">Figure 14 - Deploying application updates</p>

図 14 は、導入プロセスの概要を示しています。各アプリケーションの更新はコンテナ化されたアプリケーションとしてパッケージ化され、Amazon ECR の SaaS Boost インスタンスにアップロードされることに注意してください。

SaaS Boost が新しくアップロードされたイメージを検出すると、各テナントの環境へのローリングアップデートの一環として、該当する更新プロセスをトリガーします。これにより、DevOps パイプラインから手間のかかる作業が不要になり、SaaS Boost に責任が委ねられます。

## まとめ
このドキュメントでは、全体的な体験で使用されるビルディングブロックの概要を提供することで、SaaS Boost アーキテクチャのコア要素について説明しました。これにより、SaaS Boost エクスペリエンスの実装に使用されるコードとサービスを調査するための基礎ができたはずです。

このバージョンの SaaS Boost は、多くの SaaS 組織の開発を加速できる、規範的なオープンソース SaaS モデルの長期的開発の第一歩です。特定の移行パターンに焦点を当てましたが、ユーザーコミュニティと SaaS Boost チームが新しい機能を導入することを期待しています。この環境の進化を形作るために、コードを詳しく調べてフィードバックを提供することをお勧めします。