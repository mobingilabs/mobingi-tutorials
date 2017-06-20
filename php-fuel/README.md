# mobingiALM＋Dockerで開発環境を構築
 - Build web apps on mobingiALM with local docker env.

 - このチュートリアルについて
   - DockerとmobingiALMを使って開発作業を行うための環境作りとプロダクト環境へのリリースを学習することができます。
   - (構成図の絵)



# チュートリアル手順

0. Docker for macOS をインストール(Install docker for macOS)
1. Dockerイメージを作成する(Build docker image)
2. ローカルのDocker環境でアプリケーションを実行する(Run apps on local machine with docker env)
   - docker runからローカル環境のフォルダにマウントして開発作業する手順まで
3. mobingiALMに必要な設定を準備する
4. モビンギのdocker registryを利用する(Use mobingi private registry)
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

### 作業環境のファイル構成と取得(File structure on local env)
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


 2. サンプルのリポジトリを取得する(clone sample git repository)

    `https://github.com/mobingilabs/mobingi-tutorials.git`

    ```
    > cd /Users/[usename]/
    > git clone https://github.com/mobingilabs/mobingi-tutorials.git
    ```

### イメージを作成する(Build docker image for dev,release)

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

### 作成した開発用イメージを起動する
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

### 起動したdockerコンテナの中を確認する
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


## 3. mobingiALMに必要な設定を準備する
- このチュートリアルでは、mobingiALMで作成したdockerイメージを使いstackを起動するまで前準備について説明します。
- なお、dockerイメージの保管にはモビンギのDockerRegistryを利用する前提になります。
### AWSのアクセスキーを設定する
mobingiALMはマスターアカウントで各クラウドへの認証情報を管理する仕組みになっています。

認証情報設定からstack作成の対象にするアカウントを登録します。


（設定画像）


### mobingiALMのstackを起動するユーザーアカウントを作成する
- モビンギのDockerRegistryは、mobingiALMのユーザーのみ利用可能な仕組みになります。
- 作成したユーザーがdockerイメージをpushするためのログインアカウントになります。

1. mobingiALMのアカウントを作成する


2. モビンギのdocker registryにログインする


3. モビンギのdocker registryにイメージを登録する
   - Push mobingi private registry


### 用意したdockerイメージを使ってALMのstackを起動する

 - After all test pass, lunch ALM by using build image.

### 起動したstackにgitのソースを接続する

 - After lunch ALM stack, try to connect git repository.

## ローカル環境で修正したソースをmobingiALMにリリースする

 - local env change and push to git, ALM stack rebuild and switch new env automatically.
