+++
author = "Taku"
categories = ["Nuxt.js"]
date = "2019-08-19"
description = ""
featured = "nuxt-docker.png"
featuredalt = ""
featuredpath = "date"
linktitle = ""
title = "DockerにNuxt.js環境構築"
type = "post"
+++

自分の環境を汚すことなく、DockerでNuxt.jsの環境を構築します。

環境

- Mac Mojave 10.14.6
- Docker version 19.03.1, build 74b1e89
- docker-compose version 1.24.1, build 4667896b

最終的なディレクトリ構成
```
Root
├── docker-compose.yml
└── web
    ├── Dockerfile
    └── app
        ├── Nuxt.jsのソース
        ├── ...
```

## 準備 - Docker For Macのインストール
[こちら](https://www.docker.com/products/docker-desktop)から
Docker For Macのインストール  
※インストール方法に関しては[ドキュメント](https://docs.docker.com/docker-for-mac/install/)を参照ください。

Docker For Macをインストール後、「Docker is Running」となれば準備完了

## 作業ディレクトリを作成
```
$ mkdir -p [プロジェクト名]/web/app
$ cd [プロジェクト名]/web
```

## Dockerfileの作成
Docker上で動作させるコンテナの構成情報を記述するためのファイルを作成します。  

```
$ vim Dockerfile
-- 下記を記載

FROM node:10.11.0-alpine
 
RUN apk update &amp;&amp; \
    apk add git &amp;&amp; \
    npm install -g npm &amp;&amp; \
    npm install -g vue-cli &amp;&amp; \
    npm install -g nuxt &amp;&amp; \
    npm install -g create-nuxt-app
ENV HOST 0.0.0.0
 
WORKDIR /app

# 下記は後でコメントアウトを外します
# RUN yarn install

# EXPOSE 3000
# CMD ["yarn", "run", "dev"]
```
Dockerhubにあるnode:10.11.0-alpineをベースにnuxt.jsをインストールしたイメージを作成します。

※Dockerfileの書き方については[ドキュメント](http://docs.docker.jp/engine/reference/builder.html#id3)を参照ください。

## docker-compose.ymlの作成
Docker compose とは、複数のコンテナから成るサービスを構築・実行する手順を自動的にし、管理を容易にする機能です。

docker-compose.ymlにはDockerの起動定義を記述します。

```
$ cd ../
$ vim docker-compose.yml
--- 下記を記載
version: "3"
services:
  web: 
    build:
      context: ./web
      dockerfile: Dockerfile
    privileged: true
    volumes:
      - "./web/app:/app" 
      # exclude volumes
      - /app/node_modules
    ports:
      - 3000:3000
    container_name: "nuxt-sample" # Name the container
    tty: true
    stdin_open: true
```
※docker-compose.ymlの記述方法については[Docker docs](https://docs.docker.com/compose/compose-file/)を参照ください。

## Nuxt.jsの構築

### コンテナ作成
まずはイメージをビルドします。
```
$ docker-compose build
```

次に、コンテナを起動します。
```
$ docker-compose up -d
```

Nuxt.js環境を構築するため、Dockerコンテナ内に入ります。
```
$ docker-compose exec web sh
```

下記コマンドでNuxt.jsを展開します。  
※コンテナ内
```
# create-nuxt-app .
```
コマンド入力後、色々聞かれますのでこちらを参考に必要なものを取り入れてください。
```
create-nuxt-app v2.9.2
✨  Generating Nuxt.js project in .
? Project name nuxt-sample
? Project description sample
? Author name sample
? Choose the package manager Yarn
? Choose UI framework Bootstrap Vue
? Choose custom server framework None (Recommended)
? Choose Nuxt.js modules (Press <space> to select, <a> to toggle all, <i> to invert selection)
? Choose linting tools (Press <space> to select, <a> to toggle all, <i> to invert selection)
? Choose test framework None
? Choose rendering mode Universal (SSR)
```

Nuxt.jsの展開は完了です。  
docker-compose.ymlに書いてあるVolumesの設定により、ホストのappにもソースコードが同期されたと思います。(node_modulesは同期しないように設定しています。)

### プロジェクト起動
今回、Choose the package managerでyarnを選択したため、下記コマンドで必要パッケージをインストールします。  
※コンテナ内
```
# yarn install
```
パッケージインストール後、起動します。  
※コンテナ内
```
# yarn run dev
```
起動後、`Ctrl + P + Q`でコンテナから抜けます。

localhost:3000にアクセス！！  
```
$ open http://localhost:3000/
```

確認後、コンテナを終了します。
```
$ docker-compose down
```

## Dockerfileの修正
この環境を構築する際に必要なパッケージインストールコマンドと、docker-compose upの際にnuxt.jsを起動するコマンドを追加します。

```
$ vim web/Dockerfile
---

~省略~

# 下記は後でコメントアウトを外します
RUN yarn install

EXPOSE 3000
CMD ["yarn", "run", "dev"]
```

下記コマンドでDockerを再構築します。
```
$ docker-compose build
```

再構築後、起動します。
```
# コンテナ起動
$ docker-compose up -d

# サーバーの状態確認
$ docker-compose ps

# アクセス
$ open http://localhost:3000/
```

DockerにNuxt.js環境構築は以上です。  
以降はこの環境を第三者が利用する方法について記載しています。

## Gitからcloneして開発を行う場合
今回構築した環境を[こちらのリポジトリ](https://github.com/tkdev01/nuxt-docker)に上げておきました。
```
# コード配置
$ git clone https://github.com/tkdev01/nuxt-docker
$ cd nuxt-docker

# dockerイメージを構築する
$ docker-compose build

# コンテナを起動する
$ docker-compose up -d

# サーバーの状態確認
$ docker-compose ps

# アクセス
$ open http://localhost:3000/
```

## ビルド時にno space left on deviceエラーが発生した場合
### HDDの問題でないことを確認

`df -h`で空き容量を確認します。  

### イメージの削除
ビルドを繰り返してできた不要な領域は下記コマンドで削除
```
# コンテナをすべて削除
$ docker rm -f $(docker ps -aq)

# イメージをすべて削除
$ docker rmi -f $(docker images -q)

# ボリュームをすべて削除
$ docker volume rm -f $(docker volume ls -q)

〜個別削除は以下を参照〜

# 全コンテナ表示
$ docker ps -a
# id指定して削除する場合
$ docker rm container_id
 
# 全イメージ表示
$ docker images
# id指定して削除する場合
$ docker rmi image_id
```

## 参考
https://qiita.com/reflet/items/e7c33f84ab43ab237ee4

https://goodforthree.com/nuxt-docker/