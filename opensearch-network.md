# AWS OpenSearch 通信要件一覧

## 1. クライアントからOpenSearchへの通信

| FROM | FROM Port | TO | TO Port | Protocol | 備考 |
|------|-----------|----|---------|---------|----|
| アプリケーションサーバー | Ephemeral (1024-65535) | OpenSearch ドメインエンドポイント | 443 | HTTPS | VPCアクセス時はVPC内通信 |
| 開発者端末 | Ephemeral (1024-65535) | OpenSearch ドメインエンドポイント | 443 | HTTPS | パブリックアクセス時のみ可能 |
| Lambda関数 | Ephemeral (1024-65535) | OpenSearch ドメインエンドポイント | 443 | HTTPS | VPC Lambda設定が必要 |
| EC2インスタンス | Ephemeral (1024-65535) | OpenSearch ドメインエンドポイント | 443 | HTTPS | データ投入・検索クエリ実行 |
| Logstash/Fluentd | Ephemeral (1024-65535) | OpenSearch ドメインエンドポイント | 443 | HTTPS | ログ集約からの送信 |
| Kibana/OpenSearch Dashboards | Ephemeral (1024-65535) | OpenSearch ドメインエンドポイント | 443 | HTTPS | 管理UI経由のアクセス |

## 2. OpenSearch クラスタ内部通信

| FROM | FROM Port | TO | TO Port | Protocol | 備考 |
|------|-----------|----|---------|---------|----|
| データノード | 9300-9400 | データノード | 9300-9400 | TCP | ノード間クラスタ通信(Transport) |
| マスターノード | 9300-9400 | データノード | 9300-9400 | TCP | クラスタ状態管理 |
| データノード | 9300-9400 | マスターノード | 9300-9400 | TCP | マスター選出・状態同期 |
| マスターノード | 9300-9400 | マスターノード | 9300-9400 | TCP | マスター間通信(クォーラム) |
| データノード | 9200 | データノード | 9200 | HTTPS | REST API通信(内部) |
| ウォームノード | 9300-9400 | データノード | 9300-9400 | TCP | ウォーム/ホット間データ転送 |
| データノード | Ephemeral | データノード | Ephemeral | TCP | シャードレプリケーション |

## 3. OpenSearchからAWSサービスへの通信

| FROM | FROM Port | TO | TO Port | Protocol | 備考 |
|------|-----------|----|---------|---------|----|
| OpenSearch ノード | Ephemeral | S3 エンドポイント | 443 | HTTPS | スナップショット保存・復元 |
| OpenSearch ノード | Ephemeral | CloudWatch Logs | 443 | HTTPS | ログ送信(VPCエンドポイント推奨) |
| OpenSearch ノード | Ephemeral | CloudWatch Metrics | 443 | HTTPS | メトリクス送信 |
| OpenSearch ノード | Ephemeral | KMS | 443 | HTTPS | 暗号化キー管理 |
| OpenSearch ノード | Ephemeral | Amazon Cognito | 443 | HTTPS | 認証処理(有効時) |
| OpenSearch ノード | Ephemeral | AWS STS | 443 | HTTPS | IAMロール認証 |
| OpenSearch ノード | Ephemeral | VPC DNS (AmazonProvidedDNS) | 53 | DNS/UDP | 名前解決 |

## 4. データ投入パターン別通信

### 4-1. Kinesis Data Firehose経由

| FROM | FROM Port | TO | TO Port | Protocol | 備考 |
|------|-----------|----|---------|---------|----|
| データソース | Ephemeral | Kinesis Data Firehose | 443 | HTTPS | データストリーム送信 |
| Kinesis Data Firehose | Ephemeral | OpenSearch ドメイン | 443 | HTTPS | バッチ投入 |
| Kinesis Data Firehose | Ephemeral | S3 | 443 | HTTPS | 失敗データのバックアップ |

### 4-2. CloudWatch Logs経由

| FROM | FROM Port | TO | TO Port | Protocol | 備考 |
|------|-----------|----|---------|---------|----|
| CloudWatch Logs | Ephemeral | Lambda関数 | - | - | ログサブスクリプション |
| Lambda関数 | Ephemeral | OpenSearch ドメイン | 443 | HTTPS | ログデータ変換・投入 |

### 4-3. DMS経由(データベース移行)

| FROM | FROM Port | TO | TO Port | Protocol | 備考 |
|------|-----------|----|---------|---------|----|
| AWS DMS | Ephemeral | OpenSearch ドメイン | 443 | HTTPS | CDC/フルロードデータ投入 |

## 5. 管理・運用通信

| FROM | FROM Port | TO | TO Port | Protocol | 備考 |
|------|-----------|----|---------|---------|----|
| 管理者端末 | Ephemeral | OpenSearch Dashboards | 443 | HTTPS | Web UI アクセス |
| 管理者端末 | Ephemeral | OpenSearch API | 443 | HTTPS | REST API経由の管理操作 |
| AWS Console/CLI | Ephemeral | OpenSearch Service API | 443 | HTTPS | ドメイン管理・設定変更 |
| AWS Config | Ephemeral | OpenSearch Service API | 443 | HTTPS | コンプライアンスチェック |

## 6. バックアップ・リストア通信

| FROM | FROM Port | TO | TO Port | Protocol | 備考 |
|------|-----------|----|---------|---------|----|
| OpenSearch ノード | Ephemeral | S3 (スナップショットリポジトリ) | 443 | HTTPS | 自動/手動スナップショット |
| OpenSearch ノード | Ephemeral | S3 (スナップショットリポジトリ) | 443 | HTTPS | スナップショットからの復元 |

## 7. VPCエンドポイント経由通信(推奨構成)

| FROM | FROM Port | TO | TO Port | Protocol | 備考 |
|------|-----------|----|---------|---------|----|
| OpenSearch ノード | Ephemeral | VPCエンドポイント(S3) | 443 | HTTPS | インターネットゲートウェイ不要 |
| OpenSearch ノード | Ephemeral | VPCエンドポイント(CloudWatch) | 443 | HTTPS | プライベート通信 |
| OpenSearch ノード | Ephemeral | VPCエンドポイント(KMS) | 443 | HTTPS | セキュアな鍵管理 |

## 8. クロスクラスタレプリケーション(CCR)

| FROM | FROM Port | TO | TO Port | Protocol | 備考 |
|------|-----------|----|---------|---------|----|
| フォロワークラスタ | Ephemeral | リーダークラスタ | 443 | HTTPS | クロスリージョン/クラスタ間複製 |
| リーダークラスタ | Ephemeral | フォロワークラスタ | 443 | HTTPS | メタデータ同期 |

---

## セキュリティグループ設定例

### OpenSearchドメイン用セキュリティグループ(インバウンド)

| タイプ | プロトコル | ポート範囲 | ソース | 備考 |
|--------|-----------|-----------|--------|------|
| HTTPS | TCP | 443 | アプリケーションSG | クライアントアクセス |
| カスタムTCP | TCP | 9300-9400 | 自身のSG | クラスタ内部通信 |
| カスタムTCP | TCP | 9200 | 自身のSG | REST API内部通信 |

### OpenSearchドメイン用セキュリティグループ(アウトバウンド)

| タイプ | プロトコル | ポート範囲 | 宛先 | 備考 |
|--------|-----------|-----------|------|------|
| HTTPS | TCP | 443 | 0.0.0.0/0 または VPCエンドポイント | AWSサービスアクセス |
| カスタムTCP | TCP | 9300-9400 | 自身のSG | クラスタ内部通信 |
| DNS | UDP | 53 | VPC DNS | 名前解決 |

---

## 注意事項

1. **VPCアクセス推奨**: 本番環境ではVPCアクセスを使用し、パブリックインターネットからの直接アクセスを避けてください

2. **セキュリティグループ**: 
   - 最小権限の原則に従い、必要な通信のみ許可してください
   - クラスタ内部通信用に自己参照ルールを設定してください

3. **VPCエンドポイント**: 
   - S3、CloudWatch、KMS用のVPCエンドポイントを作成することでセキュリティを向上できます
   - インターネットゲートウェイが不要になり、通信がAWSネットワーク内に閉じます

4. **Node-to-Node暗号化**: 
   - 有効化することでクラスタ内部通信も暗号化されます(推奨)

5. **ポート範囲**: 
   - 9300-9400: Transport層の通信(クラスタ形成・データ転送)
   - 9200: HTTP/HTTPS API通信
   - 443: 外部公開用HTTPS(AWS管理のロードバランサー経由)

6. **エフェメラルポート**: 
   - クライアント側の送信元ポートは動的に割り当てられます
   - セキュリティグループではソースポートの制限は通常不要です

7. **DNS解決**: 
   - VPC内でAmazonProvidedDNS(169.254.169.253)への通信が必要です
   - Route 53 Resolver使用時も考慮してください

8. **マルチAZ構成**: 
   - 異なるAZ間の通信も発生するため、サブネットルーティングを確認してください

9. **モニタリング**: 
   - VPCフローログを有効化し、想定外の通信を検知できるようにしてください

10. **クロスリージョン**: 
    - CCR使用時は、リージョン間のデータ転送コストに注意してください