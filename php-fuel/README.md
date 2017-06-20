# mobingiALM＋DockerによるDevOps
Build web apps on mobingiALM with local docker enviroment.

 - このチュートリアルについて
   - DockerとmobingiALMを使って開発作業を行うための環境作りとプロダクト環境へのリリースを学習することができます。

 (構成図の絵)



# チュートリアル手順

必要な知識：gitの操作、githubの操作<br>
あるとよい知識：docker

0. Docker for macOS をインストール(Install docker for macOS)
1. Dockerイメージを作成する(Build docker image)
2. ローカルのDocker環境でアプリケーションを実行する(Run apps on local machine with docker env)
   - docker runからローカル環境のフォルダにマウントして開発作業する手順まで
3. mobingiALMに必要な設定を準備する
4. モビンギのdocker registryを利用する(Use mobingi docker registry)
5. mobingiALMにテスト用とリリース用のstackを作成する
    - Lunch ALM for test,release with registed docker image
6. 開発環境で修正したソースをmobingiALMに反映する
    - modify source code on local env, and updata mobingiALM



## 0.Docker for macOS をインストール(Install docker for macOS)
- 下記URLを参照の上、ご利用端末にインストールしてください。
- https://docs.docker.com/docker-for-mac/install/



## 1.Dockerイメージを作成する(Build docker image)
- このチュートリアルでは、作業に必要なdockerイメージを作成するところまでを説明します。
- dockerfileの記述内容については長くなるため省略します。
- 事前にgithubのアカウントを準備してください。

### 1.1.作業環境のファイル構成と取得(File structure on local env)
 1. 作業フォルダの場所
    - dockerイメージのフォルダをアタッチするため、Users/[username]/の下に作業フォルダを用意します。

      **file structure**
      ```
      Users/[username]/
          /mobingi-tutorials
          ※dockerイメージを作成するためのサンプルと作業環境一式
            /php-fuel
              /developer    ※ 開発作業用

              /docker       ※ docker作業用

      ```


 2. サンプルのリポジトリをフォークする(fork sample git repository)

    `https://github.com/mobingilabs/mobingi-tutorials.git`

    githubのリポジトリページから右上にあるforkをクリックします。

    (画像)

    用意したアカウントにフォークされたリポジトリが作成されるのでcloneします。

    ```
    > cd /Users/[usename]/
    > git clone https://github.com/[xxxx]/mobingi-tutorials.git
    ```
    ※ [xxxx]はフォークしたgithubアカウントIDになります。

### 1.2.イメージを作成する(Build docker image for dev,release)

  3. 開発用のイメージを作成

        設定内容 | 設定値
        --------------------|--------------
        docker image名 | tutorialdev01
        インストールモジュール| apache,curl,vim,git,zip,unzip,composer

        - dockerイメージの設定内容については以下のファイルを参照してください。
          - php-fuel/docker/7.0-dev/Dockerfile

        - dockerイメージの設定内容については以下のファイルを参照してください。
          - php-fuel/docker/conf/000-default.conf

        ```
        > cd php-fuel/docker
        > docker build -f 7.0-dev/Dockerfile -t tutorialdev01 .
        ```
        - ビルドが完了したら、`docker images`を実行してイメージを確認します。

  4. リリース用のイメージを作成

       設定内容 | 設定値
       --------------------|--------------
       docker image名 | tutorialrel01
       インストールモジュール| apache

       - dockerイメージの設定内容については以下のファイルを参照してください。
         - php-fuel/docker/7.0-release/Dockerfile

       - dockerイメージの設定内容については以下のファイルを参照してください。
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
always -v /Users/kodo/mobingi-tutorials/php-fuel/developer:/var/www/html -p 80:80 -it tutorialdev01 /run.sh
```

- `\`は、折返しで記載しています。１行で続けて入力してください。
 - `-v`は、環境に合わせて修正してください。
   - `[host-folder]:[container-folder]`になります。
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
- コマンドラインから以下を実行します。
```
docker ps -a
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

ホスト端末のブラウザから以下のURLでアクセスしてPHPINFOが表示されれば成功です。

`http://localhost/test.php`

![Webサイト確認画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/preview-website-preview-00.png)

## 3. mobingiALMに必要な設定を準備する
- このチュートリアルでは、mobingiALMで作成したdockerイメージを使いstackを起動するまで前準備について説明します。
- なお、dockerイメージの保管にはモビンギのDockerRegistryを利用する前提になります。
### 3.1.AWSのアクセスキーを設定する
mobingiALMはマスターアカウントで各クラウドへの認証情報を管理する仕組みになります。<br>
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
> docker tag tutorialdev01 registry.mobingi.com/testtest/
```

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/regist-mobingi-docker-00.png)



### 3.3.用意したdockerイメージを使ってALMのstackを起動する

 - After all test pass, lunch ALM by using build image.

1. 『3.2.mobingiALMのstackを起動するユーザーアカウントを作成する』で作成したアカウントでmobingiALMにログインします。<br>



![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/login-useraccount.png)



2. 下記の設定でstackを作成します。

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/create-stack-dev-00.png)




### 3.4.起動したstackにgitのソースを接続する

 - After lunch ALM stack, try to connect git repository.
 - 接続したgithubのソースコードは起動したホストインスタンス上に配置され、dockerイメージ側で保持しているソースコード配置場所に自動的にマウントします。

1. GitHubのプライベートリポジトリから接続します。<br>
ログインしていない場合、認証ページを経由するので利用を許可してください。

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/create-stack-github.png)

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/create-stack-github-01.png)

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/create-stack-github-02.png)

![設定の画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/create-stack-github-03.png)

2. githubを接続後、stack詳細のリソースのリンクを開き、Webサイトが表示されることを確認します。

![Webサイト確認画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/create-stack-01.png)



![Webサイト確認画面](https://raw.githubusercontent.com/wiki/mobingilabs/mobingi-tutorials/images/preview-website-preview-00.png)


## 4.ローカル環境で修正したソースをmobingiALM上にリリースする
- change source code on local env and push to git, ALM stack rebuild and switch new env automatically.

1. 起動したローカル環境でソースコードを修正して接続したgit repositoryにpushします。<br>

2. 自動的にmobingiALM側で検出して修正が反映されます。
