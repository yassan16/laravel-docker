# Colima + Docker 環境構築手順書

以下の手順に従うことで、macOS 上で Colima と Docker を利用した Laravel 開発環境を構築できます。  
すべてのコマンドに解説を付けているため、この手順書だけで導入できます。

---

## 1. 前提条件

- macOS
- Homebrew がインストールされていること  
  （未インストールの場合：`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`）

---

## 2. Colima のインストール

### コマンド
```bash
brew install colima
```

### 解説
Homebrew を使って **Colima（軽量 Docker 仮想化環境）**をインストールします。  
Colima は Mac に Docker Desktop を入れずに Docker を使える便利なツールです。

---

## 3. Docker CLI のインストール

### コマンド
```bash
brew install docker docker-compose
```

### 解説
Docker 本体（CLI）と docker-compose をインストールします。  
Docker Desktop なしで Docker を動作させるために必要です。

---

## 4. Colima の起動

### コマンド
```bash
colima start
```

### 解説
Colima を起動して、Docker が使える状態になります。  
起動時に仮想環境（VM）が自動的に作成されます。

---

## 5. Laravel 用プロジェクトフォルダの準備

### コマンド例
```bash
mkdir laravel_docker
cd laravel_docker
```

### 解説
Docker 関連ファイルや Laravel プロジェクトをまとめるフォルダを作成します。

---

## 6. docker-compose.yml の作成

以下のようなファイルを作成します。

```yaml
version: "3.9"

services:
  app:
    build:
      context: .
      dockerfile: docker/php/Dockerfile
    container_name: laravel_app
    volumes:
      - ./laravel_app:/var/www/html
    ports:
      - "8000:8000"
    environment:
      TZ: "Asia/Tokyo"

  mysql:
    image: mysql:8.0
    container_name: mysql_db
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: app
    volumes:
      - db_data:/var/lib/mysql
    ports:
      - "3306:3306"

volumes:
  db_data:
```

### 解説
- **app**コンテナ：Laravel アプリを動作させる PHP サーバ。
- **mysql コンテナ**：データベース。
- **volume（db_data）**：DB のデータ永続化（削除しない限り中身が残る）。

---

## 7. PHP Dockerfile の作成

`docker/php/Dockerfile` を作成。

```dockerfile
FROM php:8.4-fpm

RUN apt-get update && apt-get install -y     git unzip libzip-dev libicu-dev     && docker-php-ext-install pdo_mysql zip intl
```

### 解説
Laravel に必要な拡張（pdo_mysql / zip など）をインストールしています。

---

## 8. Docker コンテナの起動

### コマンド
```bash
docker compose up -d
```

### 解説
- `up`：compose ファイルの内容に従ってコンテナ起動
- `-d`：バックグラウンド実行  
  ※初回はビルドも同時に実行されます。

---

## 9. Laravel プロジェクト作成（後からでもOK）

### コマンド
```bash
cd laravel_app
composer create-project laravel/laravel .
```

### コマンド解説
- `composer create-project`  
  → 指定したパッケージ（ここでは Laravel）を新規プロジェクトとして作成する。
- `laravel/laravel`  
  → Laravel の公式テンプレート。
- `.`  
  → 現在のフォルダにインストールする指定。  
    新しいフォルダを作らずにそのまま入る。

---

## 10. Laravel 開発サーバの起動（コンテナ内）

### コマンド
```bash
docker exec -it laravel_app bash
php artisan serve --host=0.0.0.0
```

### 解説
- `docker exec -it laravel_app bash`  
  → app コンテナに入る
- `php artisan serve --host=0.0.0.0`  
  → 外部接続できるように 0.0.0.0 で起動  
    → Mac から `http://127.0.0.1:8000` でアクセスできる

---

## 11. コンテナ削除・再起動方法まとめ

### 停止
```bash
docker compose stop
```

### 完全削除（DBデータ含む）
```bash
docker compose down -v
```

### 再起動
```bash
docker compose up -d
```

---

## 12. よくある質問（FAQ）

### Q: コンテナを作り直したい時は？
A:  
```bash
docker compose down -v
docker compose up -d --build
```

### Q: 127.0.0.1 と localhost は同じ？
A: ほぼ同じ。  
どちらも自分の PC（ループバックアドレス）を指す。

---

この手順書の内容だけで、誰でも Colima + Docker + Laravel 開発環境を導入できます！
