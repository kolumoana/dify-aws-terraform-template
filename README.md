# dify-aws-terraform

Terraform template for Dify on AWS

1. VPCを作成(プライベートサブネット、パブリックサブネット 2つ)
2. terraform.tfvars.example を terraform.tfvars にコピー
3. vpc_id, private_subnet_ids, public_subnet_ids を terraform.tfvars に設定
4. redis_password, db_master_password, dify_db_password, dify_init_password を決めて terraform.tfvars に設定
5. backend.tfに記載のS3バケットを作成(名前を変更)(terraform-state)
6. terraform init
7. terraform plan
8. terraform apply -target aws_rds_cluster_instance.dify
9. Amazon RDSのクエリエディタでpostgresユーザー、postgresデータベースで以下のSQLを実行

    ```sql
    CREATE ROLE dify WITH LOGIN PASSWORD 'your-password'; # パスワードはterraform.tfvarsに設定したdify_db_password
    GRANT dify TO postgres;
    CREATE DATABASE dify WITH OWNER dify;
    ```

10. Amazon RDSのクエリエディタでpostgresユーザー、difyデータベースで以下のSQLを実行

    ```sql
    CREATE EXTENSION vector;
    ```

11. terraform apply

12. デプロイが完了したら、terraform.tfvars に設定した dify_init_password を使って dify_url にログイン

13. Redis パスワードの更新手順

1. 新しいRedisパスワードを環境変数として設定します：
   ```bash
   REDIS_PASSWORD='新しいパスワードを入力'
   ```

2. ROTATEコマンドを実行して、新旧両方のパスワードを一時的に有効にします：
   ```bash
   aws elasticache modify-replication-group \
     --replication-group-id dify \
     --auth-token ${REDIS_PASSWORD} \
     --auth-token-update-strategy ROTATE \
     --apply-immediately
   ```

3. SETコマンドを実行して、新しいパスワードのみを有効にします：
   ```bash
   aws elasticache modify-replication-group \
     --replication-group-id dify \
     --auth-token ${REDIS_PASSWORD} \
     --auth-token-update-strategy SET \
     --apply-immediately
   ```

この2段階のプロセスにより、アプリケーションのダウンタイムなしでRedisパスワードを安全に更新することができます。
