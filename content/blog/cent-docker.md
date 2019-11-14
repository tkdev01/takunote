+++
author = "Taku"
categories = ["Docker"]
date = "2019-08-19"
description = ""
featured = ""
featuredalt = ""
featuredpath = "date"
linktitle = ""
title = "CentOS7にDockerインストール"
type = "post"
draft = true
+++


## 参考
https://qiita.com/uhooi/items/fb14d99d3323bd2eee9d

https://qiita.com/uhooi/items/f8c67a9e716a226e28cd


cent7 git install
https://qiita.com/tomy0610/items/66e292f80aa1adc1161d

home/argus/damgit

# docker command

$ which docker-compose
/usr/local/bin/docker-compose

## Docker build
sudo /usr/local/bin/docker-compose build

## install packages
sudo /usr/local/bin/docker-compose run nuxt.js yarn install

## docker up
sudo /usr/local/bin/docker-compose up -d



sudo /usr/local/bin/docker-compose exec nuxt.js sh