# mtr-example-eap7.4to8.0


オリジナルのソースコードはこちらを参考にしています。
https://github.com/ssetoredhat/cloud-native-workshop-v2m1-labs


Migration Toolkit for Runtimesを使用してEAP 7.4からEAP 8.0βへ移行するのを試すために使用するリポジトリです。

## 前提準備

以下の製品を使用しています。
JBoss EAP 8.0β Release date: December 15, 2022
※zip File及びにServer Migrationをダウンロードしてください。
JBoss EAP 7.4
Migration Toolkit for Runtimes 1.1.0(MTA 6.2.5)
Red Hat build of OpenJDK 11.0.19-x64(Windows 64bit)
これらはすべてRed Hat Developersからダウンロードできます。
https://developers.redhat.com/products/eap/download
https://developers.redhat.com/products/mtr/download
https://developers.redhat.com/products/openjdk/download

※MTR1.1.0/MTA6.2.5はJava 17だと動作しないことが報告されていますのでご注意ください。


DBはpostgresqlを使用しています。
Podmanでpostgresqlを実行してください。

```
# イメージの取得
podman image pull docker.io/postgres

# 実行
podman run --name postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 -d postgres

# コンテナの終了
podman stop postgres

# コンテナの再起動
podman start postgres

# コンテナの削除
podman rm postgres

```

postgresqlのJDBCドライバを取得してローカルに保存しておく必要があります。
https://jdbc.postgresql.org/

## サーバーの設定
JBoss EAP 8.0βはJBoss Server Migration Toolを使用してEAP 7.4から設定を移行できますが、EAP 7.4の設定は自分自身で行う必要があります。
以下の通り実行してください。


JBoss EAPの起動および管理cliの起動

```
standalone.bat --server-config=standalone-full.xml
```

```
jboss-cli --connect
```

管理CLIからJDBCドライバのインストール及びにtopicの追加
```
jms-topic add --topic-address=orders --entries=[/topic/orders]

module add --name=org.postgres --resources=<PATH_TO_JDBC_DRIVER>\postgresql-XX.X.X.jar --dependencies=javax.api,javax.transaction.api
/subsystem=datasources/jdbc-driver=postgres:add(driver-module-name=org.postgres, driver-name=postgres)

data-source add --name=postgresDS --jndi-name=java:jboss/datasources/CoolstoreDS \
--driver-name=postgres \
--connection-url=jdbc:postgresql://localhost:5432/postgres \
--user-name=postgres --password=postgres
```

## 実行

JBoss Server Migration Tool/Migration Toolkit for Runtimesの使用方法は以下の記事で説明されています。
https://rheb.hatenablog.com/entry/2022/12/18/094444