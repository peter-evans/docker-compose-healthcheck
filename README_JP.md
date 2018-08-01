# docker-compose-healthcheck
[![Build Status](https://travis-ci.org/peter-evans/docker-compose-healthcheck.svg?branch=master)](https://travis-ci.org/peter-evans/docker-compose-healthcheck)

Docker Composeの[バージョン2.1のファイルフォーマット](https://docs.docker.com/compose/compose-file/compose-versioning/#version-21)が作成して以来、[healthcheck](https://docs.docker.com/compose/compose-file/#healthcheck)パラメータが導入されました。
これにより、サービスのコンテナが「healthy」（正常）であるかどうかを判断するためのチェックを構成できます。

## コンテナYを開始する前にコンテナXが正常になるまで待機する方法は？

これは一般的な問題であり、以前のバージョンのdocker-composeでは[wait-for-it](https://github.com/vishnubob/wait-for-it)や[dockerize](https://github.com/jwilder/dockerize)などのツールやスクリプトを使用する必要があります。
`healthcheck`パラメータを使うと、これらのツールやスクリプトの使用はもはや必要ではないことがよくあります。

## PostgreSQLが「healthy」（正常）になるまで待機する方法

特によく使用されるケースは、PostgreSQLなどのデータベースに依存するサービスです。
PostgreSQLコンテナが起動し、リクエストを受け入れる準備ができたら続行するという設定ができます。

以下の例では、`pg_isready`コマンドを使用してPostgreSQLが使用可能かどうかを定期的にチェックするように設定されています。[`pg_isready`コマンドの参照](https://www.postgresql.org/docs/9.4/static/app-pg-isready.html)
```yml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U postgres"]
  interval: 10s
  timeout: 5s
  retries: 5
```
チェックが成功すると、コンテナは「正常」となります。それまでは、`unhealthy`(障害)にとどまります。
healthcheckパラメータ `interval`、` timeout`、 `retries`の詳細については、[ドキュメンテーション](https://docs.docker.com/engine/reference/builder/#healthcheck)を参照してください。

それで、PostgreSQLに依存するサービスは `depends_on`パラメータで設定することができます。
```yml
depends_on:
  postgres-database:
    condition: service_healthy
```

## Kongを開始する前にPostgreSQLが正常になるまで待機

[この完全な例では](docker-compose.yml) docker-composeがオープンソースのAPIゲートウェイである[Kong](https://getkong.org/)を起動する前にPostgreSQLが「healthy」（正常）になるまで待機します。
また、もう一つの一時的なコンテナがKongのデータベース移行プロセスを完了するのを待ちます。

以下のようにテストします:
```
docker-compose up -d
```
両方のサービスが実行されるまで待ちます:
```
Creating network "docker-compose-healthcheck_default" with the default driver
Creating docker-compose-healthcheck_kong-database_1 ... done
Creating docker-compose-healthcheck_kong-migration_1 ... done
Creating docker-compose-healthcheck_kong_1           ... done
```
Kongの管理エンドポイントをクエリしてテストします:
```
curl http://localhost:8001/
```

## ライセンス

MIT ライセンス - 詳細については[LICENSE](LICENSE)ファイルを見てください
