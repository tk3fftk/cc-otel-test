# cc-otel-test

OpenTelemetry Collector (contrib) を Docker Compose で起動し、OTLP で受け取ったテレメトリを Grafana Cloud に転送するサンプル構成。

## 構成

- `docker-compose.yaml` — `otel/opentelemetry-collector-contrib` を単体で起動
- `otel-collector-config.yaml` — receivers / processors / exporters の宣言
- `.env.example` — Grafana Cloud 接続用シークレットのテンプレート

### パイプライン

- **traces**: OTLP → `grafanacloud` connector で派生 metrics 生成 → Grafana Cloud
- **metrics**: OTLP + connector → `deltatocumulative` で Cumulative 化 → Grafana Cloud
- **logs**: OTLP → Grafana Cloud
- Collector 自身の telemetry も Grafana Cloud へ送信

## セットアップ

```sh
cp .env.example .env
# .env を編集して以下を設定:
#   GRAFANA_CLOUD_OTLP_ENDPOINT  (リージョン別 OTLP gateway URL)
#   GRAFANA_CLOUD_INSTANCE_ID    (Stack ID)
#   GRAFANA_CLOUD_API_KEY        (Access Policy トークン)
#   GRAFANA_CLOUD_BASIC_AUTH_HEADER  ("Basic <base64(id:key)>")
docker compose up -d
```

## 公開ポート

| Port  | 用途                               |
| ----- | ---------------------------------- |
| 4317  | OTLP gRPC receiver                 |
| 4318  | OTLP HTTP receiver                 |
| 8888  | Collector 自身の Prometheus metrics |
| 13133 | health_check                       |
| 55679 | zPages デバッグ UI                 |

## 注意

- `contrib` イメージが必須(`basicauth` extension と `grafanacloud` connector を含むため)
- Grafana Cloud OTLP gateway は Cumulative temporality のみ受け付ける
- `.env` は `.gitignore` 済み
