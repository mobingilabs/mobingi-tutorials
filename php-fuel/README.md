# mobingiALMとDockerを使って開発環境を構築
 - Build web apps on mobingiALM with local docker env.

 - このチュートリアルについて
   - DockerとmobingiALMを使って開発作業を行うための環境作りとプロダクト環境へのリリースを体験することができます。
   - (構成図の絵)



# チュートリアル手順

 - Docker for macOS をインストール
   - Install docker for macOS
 - 開発用のDockerイメージを作成する
   - Build docker image for develop
 - ローカルのDocker環境でアプリケーションのテストを行う
   - Test apps on local machine with docker env
   - docker runからローカル環境のフォルダにマウントして開発作業する手順まで

 - モビンギのdocker registryを利用する
   - Use mobingi private registry
 - モビンギのdocker registryにイメージを登録する
   - Push mobingi private registry
 - 登録したdockerイメージを利用してmobingiALMのテスト用stackを起動する
   - Lunch ALM for test from build image
 - 登録したdockerイメージを利用してmobingiALMのリリース用stackを起動する
   - Lunch ALM for release from build image



## Docker for macOS をインストール(Install docker for macOS)
 - ref others

## 開発用のDockerイメージを作成する(Build docker image for develop)
 - サンプルのリポジトリを取得する
   - clone sample git repository

   ` https://github.com/hkdstamp/ubuntu-apache2-php7`

 - Change this Dockerfile for development
   - add vim,git,zip,unzip,composer


### ローカル環境でdockerイメージを実行して開発作業を行う

 - Run docker image on local and debug source.

### 用意したdockerイメージを使ってALMのstackを起動する

 - After all test pass, lunch ALM by using build image.

### 起動したstackにgitのソースを接続する

 - After lunch ALM stack, try to connect git repository.

## ローカル環境で修正したソースをmobingiALMにリリースする

 - local env change and push to git, ALM stack rebuild and switch new env automatically.
