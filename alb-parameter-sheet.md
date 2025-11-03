# AWS ALB(Public) 構築パラメータシート

## 1. 基本設定

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| ロードバランサー名 | Name | `alb-public-prod` | 用途に応じて命名を変更 |
| スキーム | Scheme | `internet-facing` | パブリックALBのため必須 |
| IPアドレスタイプ | IP address type | `ipv4` | IPv6が必要な場合は`dualstack`を選択 |
| リージョン | Region | `ap-northeast-1` (Tokyo) | 東京リージョン |

## 2. ネットワーク設定

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| VPC | VPC | `10.0.0.0/16` | 既存VPCを選択 |
| アベイラビリティゾーン1 | Availability Zone 1 | `ap-northeast-1a` | 最低2つのAZが必要 |
| サブネット1 | Subnet 1 | `10.0.1.0/24` (Public) | パブリックサブネットを選択 |
| アベイラビリティゾーン2 | Availability Zone 2 | `ap-northeast-1c` | - |
| サブネット2 | Subnet 2 | `10.0.2.0/24` (Public) | パブリックサブネットを選択 |

## 3. セキュリティグループ設定

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| セキュリティグループ名 | Security group name | `alb-public-sg` | 新規作成する場合 |
| インバウンドルール1 | Inbound rule 1 | Type: `HTTPS`, Port: `443`, Source: `0.0.0.0/0` | インターネットからのHTTPSアクセス |
| インバウンドルール2 | Inbound rule 2 | Type: `HTTP`, Port: `80`, Source: `0.0.0.0/0` | HTTPからHTTPSへのリダイレクト用(オプション) |
| アウトバウンドルール | Outbound rule | Type: `All traffic`, Destination: `0.0.0.0/0` | ターゲットへの通信用 |

## 4. リスナー設定

### 4.1 HTTPSリスナー (推奨)

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| プロトコル | Protocol | `HTTPS` | - |
| ポート | Port | `443` | - |
| デフォルトアクション | Default action | `Forward to target group` | ターゲットグループへ転送 |
| SSL証明書 | SSL certificate | `ABC.cert` | ACMまたはIAMから選択 |
| セキュリティポリシー | Security policy | `ELBSecurityPolicy-TLS13-1-2-2021-06` | 最新のTLS1.3対応ポリシー推奨 |

### 4.2 HTTPリスナー (オプション)

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| プロトコル | Protocol | `HTTP` | - |
| ポート | Port | `80` | - |
| デフォルトアクション | Default action | `Redirect to HTTPS` | HTTPSへリダイレクト推奨 |
| リダイレクト設定 | Redirect configuration | Port: `443`, Status code: `301` | 恒久的なリダイレクト |

## 5. ターゲットグループ設定

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| ターゲットグループ名 | Target group name | `tg-ecs-fargate-prod` | 用途に応じて命名 |
| ターゲットタイプ | Target type | `IP` | ECS Fargateの場合はIPを選択 |
| プロトコル | Protocol | `HTTP` | - |
| ポート | Port | `80` または `8080` | コンテナの公開ポートに合わせる |
| VPC | VPC | `10.0.0.0/16` | ALBと同じVPC |
| プロトコルバージョン | Protocol version | `HTTP1` | アプリケーションに応じて選択 |

## 6. ヘルスチェック設定

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| ヘルスチェックプロトコル | Health check protocol | `HTTP` | - |
| ヘルスチェックパス | Health check path | `/health` または `/` | アプリケーションのヘルスチェックエンドポイント |
| ヘルスチェックポート | Health check port | `traffic-port` | ターゲットポートと同じ |
| 正常のしきい値 | Healthy threshold | `2` | 連続成功回数 |
| 異常のしきい値 | Unhealthy threshold | `2` | 連続失敗回数 |
| タイムアウト | Timeout | `5` seconds | レスポンスタイムアウト時間 |
| 間隔 | Interval | `30` seconds | ヘルスチェック間隔 |
| 成功コード | Success codes | `200` | 複数指定可能(例: 200,201) |

## 7. 属性設定(オプション)

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| アイドルタイムアウト | Idle timeout | `60` seconds | デフォルト値、必要に応じて変更 |
| 削除保護 | Deletion protection | `Enabled` | 本番環境では有効化推奨 |
| HTTP/2 | HTTP/2 | `Enabled` | デフォルトで有効 |
| アクセスログ | Access logs | `Enabled` | S3バケットの指定が必要 |
| アクセスログS3バケット | Access logs S3 bucket | `s3://alb-logs-bucket/prefix` | ログ保存先(事前作成必要) |
| クロスゾーン負荷分散 | Cross-zone load balancing | `Enabled` | デフォルトで有効(ALB) |
| HTTP desync緩和モード | Desync mitigation mode | `defensive` | セキュリティ強化のため |

## 8. タグ設定

| キー | 値 | 備考 |
|------|-----|------|
| Name | `alb-public-prod` | 識別用 |
| Environment | `production` または `staging` | 環境識別 |
| ManagedBy | `terraform` または `manual` | 管理方法 |
| CostCenter | `[部署コード]` | コスト管理用(任意) |

## 9. 構築後の確認事項

| 確認項目 | 確認内容 | 備考 |
|---------|---------|------|
| DNS名 | ALBのDNS名を確認 | Route53等でCNAMEレコード作成に使用 |
| ヘルスチェック | ターゲットのヘルスステータスが`healthy`であること | unhealthyの場合は設定を見直し |
| HTTPS接続 | ブラウザでHTTPS接続が正常にできること | 証明書エラーがないか確認 |
| セキュリティグループ | 必要最小限のアクセスのみ許可されていること | セキュリティベストプラクティス |
| アクセスログ | ログがS3に出力されていること | 有効化した場合 |

## 10. 注意事項

- ALBは少なくとも2つの異なるAZのサブネットが必要です
- パブリックALBの場合、サブネットにインターネットゲートウェイへのルートが必要です
- ECS Fargateをターゲットとする場合、ターゲットタイプは「IP」を選択してください
- SSL証明書(ABC.cert)は事前にACMまたはIAMにインポートしておく必要があります
- ターゲットグループのポートは、ECSタスク定義のコンテナポートと一致させてください
- 本番環境では削除保護とアクセスログを有効化することを推奨します
- セキュリティグループは用途に応じて適切に制限してください