# mobingiALM＋DockerによるDevOps
〜Build web apps on mobingiALM with local docker enviroment.〜

 - このチュートリアルについて
   - DockerとmobingiALMを使って開発作業を行うための環境作りとプロダクト環境へのリリースを学習することができます。

   ![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/mobingiALM-devops.png)



# チュートリアル手順

必要な知識：gitの操作、githubの操作

あるとよい知識：docker

0. Docker for macOS をインストール
1. Dockerイメージを作成する
2. ローカルのDocker環境でアプリケーションを実行する
   - docker runからローカル環境のフォルダにマウントして開発作業する手順まで
3. mobingiALMに必要な設定を準備する
4. ローカル環境で修正したソースをmobingiALM上にリリースする
5. PHPFrameworkのFuelを導入する



## 0.Docker for macOS をインストール
- 下記URLを参照の上、ご利用端末にインストールしてください。
- https://docs.docker.com/docker-for-mac/install/



## 1.Dockerイメージを作成する
- このチュートリアルでは、作業に必要なdockerイメージを作成するところまでを説明します。
- dockerfileの記述内容については長くなるため省略します。
- 事前にgithubのアカウントを準備してください。

dockerイメージの設定内容

設定内容 | 設定値
--------------------|--------------
apache | virtualhostで実行
confパス| /etc/apache2/sites-available/000-default.conf
mountフォルダ|/configに記載されているパス（/var/www/html）

※ 設定内容の詳細については、[mobingiDocs](https://docs.mobingi.com/official/guide/jp/custom-image)も参考にしてください。


### 1.1.作業環境のファイル構成と取得
 1. 作業フォルダの場所
    - dockerイメージのフォルダをアタッチするため、/Users/[username]/の下に作業フォルダを用意します。

      **file structure**
      ```
      /Users/[username]/
          mobingi-tutorials/
          ※dockerイメージを作成するためのサンプルと作業環境一式
            php-fuel/
              developer/    ※ 開発作業用

              docker/       ※ docker作業用

      ```


 2. サンプルのリポジトリをフォークする

    `https://github.com/mobingilabs/mobingi-tutorials.git`

    githubのリポジトリページから右上にあるforkをクリックします。

    ![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/github-fork.png)

    用意したアカウントにフォークされたリポジトリが作成されるのでcloneします。

    ```
    > cd /Users/[username]/
    > git clone https://github.com/[xxxx]/mobingi-tutorials.git
    ```
    ※ [xxxx]はフォークしたgithubアカウントIDになります。

### 1.2.用途別にイメージを作成する
ここでは開発用とリリース用で設定内容の異なるイメージを作成します。

  1. 開発用のイメージを作成

        設定内容 | 設定値
        --------------------|--------------
        docker image名 | tutorialdev01
        インストールモジュール| apache,curl,vim,git,zip,unzip,composer

        - dockerイメージの設定内容については以下のファイルを参照してください。
          - php-fuel/docker/7.0-dev/Dockerfile
          -
        - Apacheの設定内容については以下のファイルを参照してください。
          - php-fuel/docker/conf/000-default.conf

        ```
        > cd php-fuel/docker
        > docker build -f 7.0-dev/Dockerfile -t tutorialdev01 .
        ```
        - ビルドが完了したら、`docker images`を実行してイメージを確認します。
        -

  2. リリース用のイメージを作成

       設定内容 | 設定値
       --------------------|--------------
       docker image名 | tutorialrel01
       インストールモジュール| apache

       - dockerイメージの設定内容については以下のファイルを参照してください。
         - php-fuel/docker/7.0-release/Dockerfile

       - Apacheの設定内容については以下のファイルを参照してください。
         - php-fuel/docker/conf/001-default.conf

       ```
       > cd php-fuel/docker
       > docker build -f 7.0-release/Dockerfile -t tutorialrel01 .
       ```
       - ビルドが完了したら、`docker images`を実行してイメージを確認します。


## 2. ローカルのDocker環境でアプリケーションを実行する
- このチュートリアルでは、作成したdockerイメージを使用してローカル環境でWebアプリケーションを実行するところまでを説明します。

### 2.1.作成した開発用イメージを起動する
- コマンドラインから以下を実行します。

```
docker run --name tutorialdev-alone --restart \
always -v /Users/kodo/mobingi-tutorials:/var/www/html -p 80:80 -d tutorialdev01
```

- `\`は、折返しで記載しています。１行で続けて入力してください。
 - `-v`の`[host-folder]`は利用環境に合わせて修正してください。
   - `[host-folder]:[container-folder]`になります。
   - `[container-folder]`は、dockerイメージのapache設定に関係するので変更しないでください。
   - ホスト側で指定できるのは、dockerの環境設定でマウント対象となるフォルダ配下になります。
 - `--restart always`は、dockerホストを再起動すると自動的に起動するオプションになります。

    ### トラブルシューティング
      - docker imageが重複して作成できない場合
      ```
      > docker images
      > docker rmi xxxxxx(イメージID)
      ```
      - docker runでnameが重複して実行できない場合
      ```
      > docker ps -a
      > docker rm `docker ps -a -q`
      ```

### 2.2.起動したdockerコンテナの中を確認する
- 新しいterminalを起動し、コマンドラインから以下を実行します。
```
docker ps
```
コンテナIDを確認します。
```
> docker exec -it [contianer id] /bin/bash
> root@[container id]:/
```

コンテナ環境からマウントしたフォルダが見えているか確認する

```
> cd /var/www/html
> ls
```

ホスト端末のブラウザから以下のURLでアクセスしてPHPInfoが表示されれば成功です。

`http://localhost/test.php`

![Webサイト確認画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/preview-website-preview-00.png)

### 2.3.作成したリリース用イメージを起動する
- コマンドラインから以下を実行します。

```
docker run --name tutorialrel-alone -v /Users/kodo/mobingi-tutorials:/var/www/html -p 80:80 -d tutorialrel01
```
**※ イメージ作成後の確認は、開発用と同様になります。<br>
※ 利用するポートが重複するため開発用イメージを停止してから起動してください。**

## 3. mobingiALMに必要な設定を準備する
- このチュートリアルでは、mobingiALMで作成したdockerイメージを使いstackを起動するまで前準備について説明します。
- なお、dockerイメージの保管にはモビンギのDockerRegistryを利用する前提になります。
### 3.1.AWSのアクセスキーを設定する
mobingiALMはマスターアカウントで各クラウドへの認証情報を管理する仕組みになります。

このため認証情報設定から、stack作成の対象にするアカウント情報（認証Key）を登録します。

![AWS認証情報設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/alm-credential-00.png)

### 3.2.mobingiALMのstackを起動するユーザーアカウントを作成する
- モビンギのDockerRegistryは、mobingiALMのユーザーのみ利用可能となっています。
- ここで作成したユーザーがdockerイメージをpushするためのログインアカウントになります。

設定内容 | 設定値
--------------------|--------------
mobingiALMのユーザー | testtest
（開発用）| ー
登録するイメージ| tutorialdev01
作成するrepository| regitry.mobingi.com/testtest/tutorialdev01
repositoryの設定| public
（リリース用）| ー
登録するイメージ| tutorialrel01
作成するrepository| registry.mobingi.com/testtest/tutorialrel01
repositoryの設定| public

※ repositoryのprivate設定は、近日のupdateで利用可能になります。

1. mobingiALMのアカウントを作成する
- マスターアカウントの一般設定からユーザー設定を開きます。

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/create-alm-useraccount.png)


2. モビンギのdocker registryにrepositoryを作成する
- 作成したユーザーアカウントでログインして、コンテナレジストリからdockerイメージを登録するrepositoryを作成します。
- repositoryの公開設定をpublicに変更します。

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/create-container-repositry-00.png)

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/create-container-registry-01.png)


3. モビンギのdocker registryにログインする
- ローカル環境から以下のコマンドを実行します。
```
> docker login registry.mobingi.com
```
※ ログインIDとパスワードは、前述『1.mobingiALMのアカウントを作成する』で作成したアカウントになります。

4. モビンギのdocker registryにイメージを登録する
- ローカル環境から以下のコマンドを実行します。
```
> docker tag tutorialdev01 registry.mobingi.com/testtest/tutorialdev01
```

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/regist-mobingi-docker-00.png)

**※ リリース用のdockerイメージも同様に作成しpushします。**


### 3.3.用意したdockerイメージを使ってALMのstackを起動する
各stackを作成するときに設定する情報

設定内容 | 設定値
--------------------|--------------
mobingiALMのユーザー | testtest
（開発用stack）| tutorialdel01-01
割当てるrepository| regitry.mobingi.com/testtest/tutorialdev01
割当てるgithub| forkしたアカウントのmobingi-tutorials
割当てるgithubのbranch| 任意で作成したブランチ
（リリース用stack）| tutorialrel01-01
割当てるrepository| registry.mobingi.com/testtest/tutorialrel01
割当てるgithub| forkしたアカウントのmobingi-tutorials
割当てるgithubのbranch| master

1. 『3.2.mobingiALMのstackを起動するユーザーアカウントを作成する』で作成したアカウントでmobingiALMにログインします。



![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/login-useraccount.png)



2. 下記の設定でstackを作成します。

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/create-stack-dev-00.png)


**※ リリース用stackも同様に作成します。**


### 3.4.起動したstackにgithubのソースを接続する

 - 接続したgithubのソースコードは起動したホストインスタンス上に配置され、dockerイメージ側で保持しているソースコード配置場所に自動的にマウントします。

1. GitHubのプライベートリポジトリから接続します。


![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/create-stack-github.png)

ログインしていない場合、認証ページを経由するので利用を許可してください。

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/create-stack-github-01.png)

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/create-stack-github-02.png)

接続が完了した設定画面

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/create-stack-github-03.png)

2. githubを接続後、stack詳細のリソースのリンクを開き、Webサイトが表示されることを確認します。

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/create-stack-01.png)

初期状態でアクセスするとPHPInfoが表示されます

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/preview-website-preview-00.png)

**※ リリース用github接続も同様に設定します。**

## 4.ローカル環境で修正したソースをmobingiALM上にリリースする
- このチュートリアルでは、ローカル環境で起動したdockerイメージの環境でソースを修正して、各stack環境に反映されることを確認するところまでを説明します。

1. 起動したローカル環境でソースコードを修正して接続したgit repositoryにpushします。

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/update-local-source.png)



2. 自動的にmobingiALM側で検出して修正が反映されます。

githubへpushするとmobingiALMのstack側で自動的にソースコードを取得し環境を入替えます。
![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/update-alm-waiting.png)

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/update-alm-done.png)

環境の切替が完了すると反映された修正が確認できます。

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/update-alm-website.png)


## 5.PHPFrameworkのFuelPHPを導入する
- このチュートリアルでは、PHPFrameworkのFuelPHPを構築した環境に導入するところまでを説明します。

1. ローカル開発環境上でFuelを設定する

    http://fuelphp.jp/docs/1.9/

    こちらの手順をもとにローカル環境のフォルダ上で実行します。

    ソースコードの開発作業フォルダまで移動します。

    ```
    cd /Users/xxx/xxxx/mobingi-tutorials/fuel-php/developer

    curl https://get.fuelphp.com/oil | sh
    mkdir Sites
    cd Sites
    oil create blog
    ```
    composerから必要なモジュールを取得してfuelPHPを実行するフォルダを生成します。

    ![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/fuelphp-install-01.png)

    ![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/fuelphp-install-02.png)

    ![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/fuelphp-install-03.png)

    ![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/fuelphp-install-04.png)


    http://localhost/Sites/blog/public

    ローカル開発環境上で記URLからFuelPHPのデモサイトが確認できたら完了です。




2. 開発用のgithubにsource code一式をアップする

    次に作成したデモサイトをgithubにアップします。

    アップする際にfuelPHPの関連ソースを全て追加します。

    ```
    cd /Users/xxx/xxxx/mobingi-tutorials/fuel-php/developer

    git add Sites
    git commit -m 'update fuel content'

    git add -f Sites/blog/fuel/core/
    git add -f Sites/blog/fuel/package/
    git add -f Sites/blog/fuel/vendor/
    touch Sites/blog/fuel/app/logs/.gitkeep
    touch Sites/blog/fuel/app/cache/.gitkeep
    touch Sites/blog/fuel/app/tmp/.gitkeep

    echo 'init'>Sites/blog/fuel/app/logs/.gitkeep
    echo 'init'>Sites/blog/fuel/app/cache/.gitkeep
    echo 'init'>Sites/blog/fuel/app/tmp/.gitkeep

    git add -f Sites/blog/fuel/app/logs/.gitkeep
    git add -f Sites/blog/fuel/app/cache/.gitkeep
    git add -f Sites/blog/fuel/app/tmp/.gitkeep

    git commit -m 'update fuel core'
    git push
    ```


3. log、cache、tmpのパーミッションを修正するスクリプト

   環境更新時、gitからソースフォルダに含む動的なファイル出力先のパーミッションがない状態になるので、mobingi-install.shを用意して対応します。

   mobingi-install.shはgitのリポジトリのルート直下に配置します。


   ```
   chmod -R 777 /var/www/html/php-fuel/developer/Sites/blog/fuel/app/logs
   chmod -R 777 /var/www/html/php-fuel/developer/Sites/blog/fuel/app/tmp
   chmod -R 777 /var/www/html/php-fuel/developer/Sites/blog/fuel/app/cache
   ```




5. リリース用のgithubにマージする

    github上で開発環境に接続しているブランチからmasterブランチへマージします。


5. fuelPHPが動作していることを確認する

    スタック詳細の右上にあるリソースのリンクを開くきます。

    http://[stackのurl]/Sites/blog/public/




### 補足

FuelPHPの場合、PublicフォルダをDocumentRootにする想定となるので、今回の手順ではdockerイメージに含むapacheの設定を修正する必要があります。

また、ログやキャッシュなど動的に出力されるファイルがdockerコンテナ上に出力されるため、必要に応じて永続化する対応が必要になります。

この点については、今後のmobingiALMのアップデートで対応できるよう検討中です。

以上で、本チュートリアルは終了となります。
