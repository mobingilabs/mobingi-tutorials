# Build web apps on mobingiALM with local docker env.
# mobingiALMとDockerを使って開発環境を構築

 - このチュートリアルについて
   - DockerとmobingiALMを使って開発作業を行うための環境作りとプロダクト環境へのリリースを体験することができます。
   - (構成図の絵)



# チュートリアル手順

 - Docker for macOS をインストール
 - Install docker for macOS
 - 開発用のDockerイメージを作成する
 - Build docker image for develop
 - ローカルのDocker環境でアプリケーションのテストを行う
 - Test apps on local machine　with docker env
 - モビンギのdocker registryを利用する
　　- Use mobingi private registry
 - モビンギのdocker registryにイメージを登録する
 - Push mobingi private registry
 - 登録したdockerイメージを利用してmobingiALMのテスト用stackを起動する
 - Lunch ALM for test from build image
 - 登録したdockerイメージを利用してmobingiALMのリリース用stackを起動する
 - Lunch ALM for release from build image
 


## Install docker for macOS

## Build docker image

 - clone sample git repository

` https://github.com/hkdstamp/ubuntu-apache2-php7`

 - Change this Dockerfile for development
   - add vim,git,zip,unzip,composer
 
 
### build docker image on local.
 
 - Run docker image on local and debug source.
 - After all test pass, lunch ALM by using build image.

 - After lunch ALM stack, try to connect git repository.

### above done, it is ready for development throght git.

 - local env change and push to git, ALM stack rebuild and switch new env automatically.
