# AWS Aurora PostgreSQL Serverless v2 構築パラメータシート

## 基本情報

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| エンジンタイプ | Engine type | Amazon Aurora PostgreSQL-Compatible Edition | - |
| エンジンバージョン | Engine version | PostgreSQL 17.4 (最新推奨) | (2025/11/3時点)、利用可能な最新バージョンを選択 |
| テンプレート | Template | Production / Dev/Test | 本番環境はProduction推奨 |

## DBクラスター設定

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| DBクラスター識別子 | DB cluster identifier | [任意の名前] | 例: myapp-aurora-cluster-prod |
| マスターユーザー名 | Master username | [任意のユーザー名] | デフォルト: postgres |
| マスターパスワード | Master password | [強固なパスワード] | 最小8文字、英数字・記号を含む |
| データベース名 | Database name | [任意のDB名] | 初期データベース名（オプション） |

## ネットワーク設定

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| VPC | Virtual Private Cloud | 10.0.0.0/16 | 既存VPCを選択 |
| DBサブネットグループ | DB subnet group | 新規作成または既存選択 | 異なるAZのサブネットが必要 |
| サブネット1 | Subnet 1 | 10.0.1.0/24 (ap-northeast-1a) | プライベートサブネット推奨 |
| サブネット2 | Subnet 2 | 10.0.2.0/24 (ap-northeast-1c) | プライベートサブネット推奨 |
| パブリックアクセス | Public access | No | セキュリティ上、通常はNo推奨 |
| VPCセキュリティグループ | VPC security group | [既存または新規作成] | PostgreSQLポート5432の適切な制御 |

## キャパシティ設定（Serverless v2）

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| キャパシティタイプ | Capacity type | Serverless v2 | 必ずServerless v2を選択 |
| 最小ACU | Minimum ACUs | 0.5 | 最小値は0.5 ACU（1 ACU = 2GB RAM） |
| 最大ACU | Maximum ACUs | 16 | 必要に応じて調整（最大128 ACU） |

## インスタンス設定

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| Writerインスタンス | Writer instances | 1 | 常に1台（プライマリ） |
| Readerインスタンス数（最小） | Reader instances (min) | 0 | Auto Scaling最小値 |
| Readerインスタンス数（最大） | Reader instances (max) | 5 | Auto Scaling最大値、必要に応じて調整 |
| DBインスタンスクラス | DB instance class | db.serverless | Serverless v2の場合は自動設定 |

## Auto Scaling設定（Readerインスタンス）

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| Auto Scaling有効化 | Enable Auto Scaling | 有効 | Readerインスタンスの自動スケーリング |
| ポリシー名 | Policy name | [任意の名前] | 例: aurora-reader-autoscaling |
| ターゲットメトリクス | Target metric | Average CPU utilization | または Average connections |
| ターゲット値 | Target value | 70% | CPU使用率の目標値（推奨: 70-80%） |
| スケールインクールダウン | Scale in cooldown | 300秒 | インスタンス削除までの待機時間 |
| スケールアウトクールダウン | Scale out cooldown | 300秒 | インスタンス追加までの待機時間 |

## バックアップ設定

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| バックアップ保持期間 | Backup retention period | 7日 | 本番環境は7日以上推奨（最大35日） |
| バックアップウィンドウ | Backup window | 自動 または 指定時間 | 例: 18:00-19:00 UTC（日本時間 03:00-04:00） |
| Backtrackの有効化 | Enable Backtrack | 有効/無効 | ポイントインタイムリカバリの高速化 |
| バックアップの暗号化 | Backup encryption | 有効 | AWS KMS による暗号化 |

## メンテナンス設定

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| メンテナンスウィンドウ | Maintenance window | 自動 または 指定時間 | 例: 月曜 19:00-20:00 UTC |
| マイナーバージョン自動アップグレード | Auto minor version upgrade | 有効/無効 | 本番環境では慎重に判断 |

## モニタリング設定

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| 拡張モニタリング | Enhanced monitoring | 有効 | きめ細かなメトリクス取得 |
| モニタリングロール | Monitoring role | rds-monitoring-role | IAMロールの自動作成または既存選択 |
| モニタリング粒度 | Granularity | 60秒 | 1秒、5秒、10秒、15秒、30秒、60秒から選択 |
| Performance Insights | Performance Insights | 有効 | パフォーマンス分析に有用 |
| Performance Insights保持期間 | Retention period | 7日 | 無料枠は7日、長期保存は有料 |

## セキュリティ設定

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| 暗号化 | Encryption | 有効 | AWS KMS による保存時暗号化 |
| KMSキー | KMS key | aws/rds または カスタムキー | 本番環境ではカスタムキー推奨 |
| 削除保護 | Deletion protection | 有効 | 本番環境では必ず有効化 |
| IAMデータベース認証 | IAM database authentication | 有効/無効 | パスワードレス認証が可能 |

## ログエクスポート設定

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| PostgreSQLログ | PostgreSQL log | 有効 | CloudWatch Logsへのエクスポート |
| Upgradeログ | Upgrade log | 有効 | アップグレード時のログ |

## タグ設定

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| タグ - Name | Key: Name | Value: [クラスター名] | リソース識別用 |
| タグ - Environment | Key: Environment | Value: Production/Staging/Dev | 環境識別 |
| タグ - Project | Key: Project | Value: [プロジェクト名] | コスト管理用 |
| タグ - Owner | Key: Owner | Value: [担当者/チーム名] | 管理責任者 |

## 追加オプション

| 項目 | パラメータ名 | 設定値 | 備考 |
|------|------------|--------|------|
| DBクラスターパラメータグループ | DB cluster parameter group | default.aurora-postgresql15 | カスタム設定が必要な場合は新規作成 |
| DBパラメータグループ | DB parameter group | default.aurora-postgresql15 | カスタム設定が必要な場合は新規作成 |
| オプショングループ | Option group | default:aurora-postgresql-15 | 通常はデフォルトで問題なし |
| フェイルオーバー優先度 | Failover priority | tier-0 から tier-15 | 数字が小さいほど優先度が高い |

## 注意事項

### 重要なポイント
- **Serverless v2の要件**: クラスター内の全インスタンスがServerless v2である必要があります
- **ACU設定**: 最小ACUは0.5、最大は128まで設定可能。ワークロードに応じて適切に設定してください
- **Reader Auto Scaling**: CPU使用率やコネクション数をベースに自動スケーリングが可能
- **コスト**: ACU単位での課金となり、使用した分だけ支払う従量課金制
- **マルチAZ構成**: 高可用性のため、異なるAZにサブネットを配置することが必須

### セキュリティ推奨事項
- マスターパスワードは強固なものを設定し、AWS Secrets Managerでの管理を推奨
- 本番環境では必ず削除保護を有効化
- VPCセキュリティグループで必要最小限のアクセスのみ許可
- 保存時暗号化と転送中暗号化（SSL/TLS）の両方を有効化

### 運用推奨事項
- Performance Insightsを有効化してパフォーマンスを継続的に監視
- CloudWatch Alarmsで重要なメトリクスにアラートを設定
- 定期的なバックアップテストとリストア訓練の実施
- 本番環境へのデプロイ前にステージング環境での十分なテスト実施