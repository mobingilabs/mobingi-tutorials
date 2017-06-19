# mobingiALMとDockerを使って開発環境を構築
 - Build web apps on mobingiALM with local docker env.

 - このチュートリアルについて
   - DockerとmobingiALMを使って開発作業を行うための環境作りとプロダクト環境へのリリースを体験することができます。
   - (構成図の絵)



# チュートリアル手順

 - 0.Docker for macOS をインストール
   - Install docker for macOS
 - 1.開発用のDockerイメージを作成する
   - Build docker image for develop
 - 1.ローカルのDocker環境でアプリケーションのテストを行う
   - Test apps on local machine with docker env
   - docker runからローカル環境のフォルダにマウントして開発作業する手順まで

 - 2.モビンギのユーザーアカウントを用意する
 - 2.モビンギのdocker registryを利用する
   - Use mobingi private registry
 - 2.モビンギのdocker registryにイメージを登録する
   - Push mobingi private registry
 - 3.登録したdockerイメージを利用してmobingiALMのテスト用stackを起動する
   - Lunch ALM for test from build image
 - 3.登録したdockerイメージを利用してmobingiALMのリリース用stackを起動する
   - Lunch ALM for release from build image
 - 4.開発環境で修正したソースをmobingiALMに反映する
     - modify source code on local env, and updata mobingiALM



## Docker for macOS をインストール(Install docker for macOS)
 - https://docs.docker.com/docker-for-mac/install/



## 作業環境のファイル構成(File structure on local env)
 - 作業フォルダの場所

 - dockerイメージのフォルダをアタッチするため、Users/[username]/の下に作業フォルダを用意します。

   - file structure
   `
   Users/[username]/
     workfolder/
       /[docker-work-folder]
       ※dockerイメージを作成するためのサンプル
       /[webapp-work-folder]
   `
## 開発用のDockerイメージを作成する(Build docker image for develop)
 - サンプルのリポジトリを取得する
   - clone sample git repository

   `https://github.com/mobingilabs/mobingi-tutorials.git`

 - Change this Dockerfile for development
   - add vim,git,zip,unzip,composer
     - ref Dockerfile

   - setup apache conf
    - conf/000-default.conf

   `
   # cd php-fuel/docker
   # docker build -f 7.0-dev/Dockerfile -t tutorialdev01 .
   `

   - Change this Dockerfile for release
     - remove vim,git,zip,unzip,composer
       - ref Dockerfile

     - setup apache conf
      - conf/001-default.conf

     `
     # cd php-fuel/docker
     # docker build -f 7.0-release/Dockerfile -t tutorialrel01 .
     `


### ローカル環境でdockerイメージを実行して開発作業を行う

 - Run docker image on local and debug source.

 `
 Docker run --name mobingi-alm-alone --restart always -v /Users/kodo/Develops/hkdstamp:/var/www/html --env-file /Users/kodo/Develops/hkdstamp/env/envlist -p 80:80 -p 443:443 -it modev01 /run.sh
 `


### 用意したdockerイメージを使ってALMのstackを起動する

 - After all test pass, lunch ALM by using build image.

### 起動したstackにgitのソースを接続する

 - After lunch ALM stack, try to connect git repository.

## ローカル環境で修正したソースをmobingiALMにリリースする

 - local env change and push to git, ALM stack rebuild and switch new env automatically.
