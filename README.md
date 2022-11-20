# elasticsearch_hands_on_M1

# Using Docker

## ElasticSearchコンテナ作成

1. ElasticSearchイメージの取得
```bash
$ docker pull elasticsearch:8.3.1
```

2. Dockerネットワークの構築
```bash
docker network create es_net
```

3. ElasticSearchコンテナの作成
```bash
$ docker run -d --name elasticsearch --net es_net -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:8.3.1
```

## Kibanaコンテナ作成

1. Kibanaイメージの取得
```bash
$ docker pull kibana:8.3.2
```

2. Kibanaコンテナ作成
```bash
$ docker run -d --name kibana --net es_net -p 5601:5601 kibana:8.3.2
```

## コンテナアクセス

1. ローカルホストにアクセス
```
http://127.0.0.1:5601/
```

Elastic設定画面に遷移するが、tokenを問われる。
tokenがわからないのでtokenを確認する。

2. トークンの確認
```bash
# go into console container
$ docker exec -it elasticsearch /bin/sh

# create enrollment token
$ bin/elasticsearch-create-enrollment-token --scope kibana
```

tokenが生成されるので、それをコピペ。

3. 認証画面コードの確認
トークンを貼り付け後の遷移画面で、認証コードを問われる。
```bash
# go into console container
$ docker exec -it kibana /bin/sh

# get kibana verification code
$ bin/kibana-verification-code
```
6桁の認証コードを画面に貼り付け、認証させる。

4. ユーザー認証
ログインユーザー情報を問われる。
ユーザー情報を知らないのので確認する。
```bash
# go into console container
$ docker exec -it elasticsearch /bin/sh

# reset password for elasticsearch
$ bin/elasticsearch-reset-password -a -u elastic
```

5. ローカルホスト接続
Dockerコンテナからhttp_ca.crtセキュリティ証明書をローカルマシンにコピーする。
```bash
$ docker cp elastic:/usr/share/elasticsearch/config/certs/http_ca.crt .
```

Dockerコンテナからコピーしたhttp_ca.crtファイルを使って、認証を行い、Elasticsearchクラスタに接続できることを確認します。
プロンプトが表示されたら、elastic userのパスワードを入力します。

```bash
curl --cacert http_ca.crt -u elastic https://localhost:9200
```