# AWS Cloud9 を使用して AWS SaaS Boost をインストールする方法

SaaS Boost インストーラを実行するために必要な前提条件がローカルマシンにインストールされていない場合は、[AWS Cloud9](https://aws.amazon.com/cloud9/) を使用するとすばやく簡単に開始できます。

デフォルトでは、Cloud9 は AWS コンソールにログインしたユーザーまたはロールの AWS 権限を継承します。そのロールに AWS SaaS Boost のインストールに必要な完全な管理者権限があることを確認してください。

1.	AWS コンソールにログインし、Amazon Linux 2 と 4 GB 以上の RAM を搭載したインスタンスタイプを使用して AWS Cloud9 環境を作成します。たとえば、**Other** インスタンスタイプを選択し、**T2 medium** または **T3 medium** を選択します。

2.	**IDEを開く** ボタンをクリックして Cloud9 環境を起動します。

3.	Cloud9 環境には、デフォルトで 10 GB のディスク容量があります。これは SaaS Boost インストーラを実行するのに十分なディスク容量ではない可能性があります。この [resize.sh シェルスクリプト](https://docs.aws.amazon.com/cloud9/latest/user-guide/move-environment.html#move-environment-resize) を使用して Cloud9 環境のサイズを変更します (少なくとも 20 GB を推奨)。スクリプトの内容を Cloud9 インスタンスの新しいファイルにコピーし、実行します。
```
sh resize.sh 20
```

4.	以下のコマンドを実行して SaaS Boost Git リポジトリをクローンします。
```
cd ~/environment
git clone https://github.com/awslabs/aws-saas-boost.git ./aws-saas-boost
```
5.	デフォルトよりも新しいバージョンの Apache Maven が必要です。より新しいバージョンをインストールするには、以下のコマンドを実行します。
```
sudo wget http://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo
sudo sed -i s/\$releasever/6/g /etc/yum.repos.d/epel-apache-maven.repo
sudo yum install -y apache-maven
```

6.	デフォルトの Java バージョンを Amazon Corretto 11 に戻します (両方についてオプション 1 を選択してください)
```
sudo alternatives --config java

sudo alternatives --config javac
```

7. サンプルアプリを実行する予定の場合は、サンプルビルドスクリプトで使用する `jq` をインストールする必要があります
```
sudo yum install jq
```

8.	AWS SaaS ブーストインストールスクリプトを実行します
```
cd ~/environment/aws-saas-boost
./install.sh
```

9. インストールスクリプトのプロンプトに従って AWS SaaS Boost をインストールしてください！
