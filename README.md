# dify-aws-terraform-template

AWS環境にDifyをデプロイするためのTerraformテンプレート

## 前提条件

- AWS CLIのインストールと設定
- Terraformのインストール
- AWS環境へのアクセス権限

## デプロイ手順

### 1. 初期設定

1. VPCを作成（プライベートサブネット、パブリックサブネット各2つ）
2. `terraform.tfvars.example` を `terraform.tfvars` にコピー
3. 以下の値を `terraform.tfvars` に設定:
   - `vpc_id`
   - `private_subnet_ids`
   - `public_subnet_ids`
   - `redis_password`
   - `db_master_password`
   - `dify_db_password`
   - `dify_init_password`
4. `backend.tf` に記載のS3バケットを作成（terraform-state）

### 2. Terraformの実行

1. `terraform init`
2. `terraform plan`
3. `terraform apply -target aws_rds_cluster_instance.dify`

### 3. データベースの設定

1. Amazon RDSのクエリエディタで以下のSQLを実行（postgresユーザー、postgresデータベースで実行）:
```sql
CREATE ROLE dify WITH LOGIN PASSWORD 'your-password'; -- パスワードはterraform.tfvarsに設定したdify_db_password
GRANT dify TO postgres;
CREATE DATABASE dify WITH OWNER dify;
```

2. 続いて以下のSQLを実行（postgresユーザー、difyデータベースで実行）:
```sql
CREATE EXTENSION vector;
```

### 4. デプロイの完了

1. `terraform apply`を実行
2. デプロイ完了後、`terraform.tfvars`に設定した`dify_init_password`を使って`dify_url`にログイン

### 5. migration設定を無効化

1. `terraform.tfvars`の`migration_enabled`を`false`に設定してください。
2. `terraform apply`を実行

## 本番環境の設定

### ドメインとHTTPS化

現在の構成では、環境依存性を考慮してHTTPのみ対応しています。本番環境での利用には以下の設定を推奨します：

1. ドメインの設定
   - Route 53またはDNSサービスでALBのDNS名に対するAレコードを設定

2. HTTPS化
   - AWS Certificate Manager (ACM)でSSL/TLS証明書を取得
   - ALBにHTTPSリスナー（ポート443）を追加
   - HTTPSリスナーにACM証明書を関連付け
   - 必要に応じてHTTPからHTTPSへのリダイレクトを設定

3. 環境変数の更新
   以下の環境変数をhttpsスキームに更新：
   - CONSOLE_WEB_URL
   - CONSOLE_API_URL
   - SERVICE_API_URL
   - APP_WEB_URL
   - CONSOLE_API_URL
   - APP_API_URL
