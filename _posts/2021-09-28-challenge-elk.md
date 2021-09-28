---
layout: post
title: ElasticSearch，Logstash，Kibanaを理解する
subtitle: その1 
tags: [elasticsearch, logstash, kibana]
comments: false
---

## はじめに

データを可視化することが好きなので，ElasticSearch，Logstash，Kibana（まとめてELK）に挑戦しました．

Dockerを利用すると簡単にELKを利用できるようなので，今回はDocker-Composeを利用してそれぞれのコンテナを立ててみました．

しかし1つ1つの理解が難しい；；

なので，ここでは自分が分かるように曲解したELKの説明をしますのでご容赦ください．

また，参考にしたWebサイトを載せるべきですが，今回たくさんのサイトを拝見したので省略致します．

## ELKの説明
私の頭の中のELKは以下のような構成です．矢印`-->`はログ（データ）の流れを表しています．
```
[ログがあるサーバ] --> Logstash --> ElasticSearch(データベース) --> Kibana
```


### Logstash
- 「様々な場所(サーバ)にある様々なログを収集 -> 収集したログを加工 -> ElasticSearchに送信」するサービス
- ログを収集(**Input Plugin**)して,加工(**Filter Plugin**)して,送信(**Output Plugin**)するまでの一連の処理を「**pipeline**」という

#### ★ Logstashを起動するのに必要な設定ファイル
Logstashの設定ファイルの種類は以下の2種類に大きく分けられます．
1. pipelineの設定ファイル[必要]
2. Logstashの設定ファイル[任意（デフォルトの設定でいいなら用意しなくて良い）]

以下の例のように，それぞれを配置するディレクトリを作成します．
```
作業ディレクトリ/
 |--- logstash/ <= logstash用のファイルを格納するディレクトリ（これを用意せず，作業ディレクトリに直接pipeline/とconfig/を作っても良い）
        |--- pipeline/ <= pipelineの設定ファイルを格納するディレクトリ
		|--- config/ <= Logstashの設定ファイルを格納するディレクトリ
```
　　　
##### ★★ pipelineの設定ファイル
ログを**収集(Input Plugin)**して,**加工(Filter Plugin)**して,**送信(Output Plugin)**するまでの一連の処理を記述します．
この設定ファイルの名前は「(適当な名前).conf」のように拡張子を`.conf`にする必要があります．

pipeline設定ファイルの例
```
input { # ログの収集
  file {
    path => "/var/log/nginx/access.log" # 収集するログファイルのパス
    start_position => beginning # 該当ファイルのどこから読むか
  }
}

filter { # 収集したログの加工
  grok { # ログからのキーワード抽出や変換処理
      match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
  date { # タイムスタンプの生成
      match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
      locale => "en"
  }
  mutate { # 値の変換
      replace => { "type" => "nginx_log" }
  }
}

output { # 加工したログの送信
  elasticsearch { # ログの送信先であるElasticsearchサーバの情報
    hosts => ["localhost"] # ホスト名
    index => "nginx_log" # Elasticsearchで使われるindexパターン（下で説明）
  }
}
```
indexは後に説明するElasticSearchに保存するデータのテーブルのことです．
そして，indexパターンは複数のindexの名前に一致するようなワイルドカードを含めた文字列です．
例えば，「accesslog-2015.05.12」と「accesslog-2015.05.30」というindexがあった場合，この２つに一致するindexパターンは「accesslog-2015.05*」です．


##### ★★ Logstashの設定ファイル
ログ収集時のオプションなど，Logstashの動作に関する設定をするファイルであり，例として以下のものがあります．
- [logstash.yml](https://www.elastic.co/guide/en/logstash/current/logstash-settings-file.html#logstash-settings-file): Logstashの設定フラグ
- [pipeline.yml](https://www.elastic.co/guide/en/logstash/current/multiple-pipelines.html#multiple-pipelines): 1つのLogstashインスタンスで複数のパイプラインを実行するための設定
詳しい説明および他の設定ファイルは[こちら](https://www.elastic.co/guide/en/logstash/current/config-setting-files.html#settings-files)に説明があります．



### Elasticsearch
Logstashから受け取ったデータを保存するデータベースです．


### Kibana
Elasticsearchから受け取ったデータの検索,可視化,分析を行うためのUIを提供するWebアプリケーションです．
なので私は，ElasticsearchをWebUIで操作するツールというイメージを持っています．


## Docker-ComposeによるELKの起動

### 1. docker-compose.ymlの作成
公式が[kibanaのdockerコンテナ](https://www.elastic.co/guide/en/kibana/current/docker.html),[elasticsearchのdockerコンテナ](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html),[logstashのdockerコンテナ](https://www.elastic.co/guide/en/logstash/current/docker.html)作成方法を載せているので，それらを参考にdocker-compose.ymlを作成しました．
elasticsearchの`environment`の部分に書かれているオプションは[こちら](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html)に説明があります．


```
version: '3'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.0
    container_name: elasticsearch
    ports:
      - 9200:9200
    environment:
      - discovery.type=single-node
      - node.master=true
      - node.data=true
      - bootstrap.memory_lock=true
    volumes:
      - esdata:/usr/share/elasticsearch/data
    ulimits:
      memlock:
        soft: -1
        hard: -1
    networks:
      - elastic

  kibana:
    image: docker.elastic.co/kibana/kibana:7.15.0
    container_name: kibana
    ports:
      - 5601:5601
    environment:
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200
    networks:
      - elastic

  logstash:
    image: docker.elastic.co/logstash/logstash:7.15.0
    container_name: logstash
    volumes:
      - ./logstash/pipeline/:/usr/share/logstash/pipeline/
#      - ./logstash/config/:/usr/share/logstash/config/ <= logstashの設定が必要ならコメントアウトを外し，ローカル環境にこのディレクトリと設定ファイルを用意する
    networks:
      - elastic

volumes:
  esdata:
    driver: local

networks:
  elastic:
    driver: bridge

```
### 2. Logstashの設定ファイルの作成
作業用ディレクトリに`./logstash/pipeline/`ディレクトリを作成します．
```
$ mkdir -p logstash/pipeline/
```
pipelineディレクトリの下に設定ファイルを作成します．`.conf`という拡張子であればファイル名はなんでも良いです．
例としてtest.confを示します．こちらは「5秒おきにmessageをLogstashに送信する」[公式サンプル](https://github.com/elastic/logstash-docker/blob/master/examples/logstash.conf)を参考にしました．
```
input {
  heartbeat {
    interval => 5
    message  => 'Hello from Logstash'
  }
}

filter {
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200/"]
  }
}
```

### 3. 起動
```
$ sudo docker-compose up -d
```
`docker ps`コマンドですべてのコンテナが動いていることを確認します．
Elasticsearchは[http://localhost:9200](http://localhost:9200)，Kibanaは[http://localhost:5601](http://localhost:5601)にブラウザアクセスすることでも確認できます．
いずれかのコンテナが動作していなければ，`docker logs <コンテナ名>`で原因を調べます．

### 4. Kibanaの利用
1. ブラウザからlocalhost:5601にアクセスします
2. 初回アクセス画面が出るので[Add data]ボタンを押します
3. 左メニューの[Analytics]から[Discover]を選択します
4. 下部にある[create an index pattern]を選択します
5. Create index patternが表示されるので，Nameは「*」を指定し，Timestamp fieldは適当に選択します
6. [Create index pattern]を押すと，Logstashから受け取ったデータが表示されます


## 今回はここまで
力尽きました👼
