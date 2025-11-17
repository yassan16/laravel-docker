# Colima + Docker 開発環境構築手順書  
（Laravel / Nginx / MySQL 対応・図解付き）

---

## 📘 目次

1. [概要](#概要)
2. [全体構成図](#全体構成図)
3. [事前準備](#事前準備)
4. [Colima のインストールと起動](#colima-のインストールと起動)
5. [Docker プロジェクト構成](#docker-プロジェクト構成)
6. [Dockerfile 解説](#dockerfile-解説)
7. [docker-compose.yml 解説](#docker-composeyml-解説)
8. [環境変数（.env）設定](#環境変数env設定)
9. [コンテナの起動方法](#コンテナの起動方法)
10. [Laravel プロジェクト作成手順](#laravel-プロジェクト作成手順)
11. [MySQL 接続方法（TablePlus）](#mysql-接続方法tableplus)
12. [便利コマンド集](#便利コマンド集)
13. [トラブルシュート](#トラブルシュート)

---

## 概要

本手順書は **Colima + Docker** を使い、  
**Nginx / PHP-FPM（Laravel）/ MySQL** でローカル開発環境を構築するための完全ガイドです。

インフラ初心者でもセットアップできるよう、図解・解説を含めています。

---

## 全体構成図

以下は Docker 環境全体のイメージ図です。

```mermaid
graph TD;
  A[Host OS (macOS)] --> B[Colima VM];
  B --> C[Docker Engine];
  C --> D[nginx コンテナ];
  C --> E[php-fpm(Laravel) コンテナ];
  C --> F[mysql コンテナ];

  E <-->|FastCGI| D;
  E -->|DB接続| F;

  D -->|Port 8080| A;
  F -->|Port 3306| A;
```

---

## 事前準備

### 必要ソフト

| ツール | 用途 |
|-------|------|
| Homebrew | パッケージ管理 |
| Colima | macOS の Docker 実行用 VM |
| Docker CLI | Docker コマンド |

---

## Colima のインストールと起動

### ① インストール

```sh
brew install colima
```

### ② 起動

```sh
colima start
```

※ Docker Desktop は不要です。

---

## Docker プロジェクト構成

```
laravel_docker/
│── docker/
│    ├── php/
│    │    └── Dockerfile
│    └── nginx/
│         └── default.conf
│
│── docker-compose.yml
│── .env
│── src/ ← Laravel を後から作成する場所
```

Laravel プロジェクトは **src/** にマウントします。

---

## Dockerfile 解説

php-fpm コンテナの Dockerfile 例とポイントを含みます（省略: 本文に挿入可）

---

## docker-compose.yml 解説

Nginx / PHP-FPM / MySQL の 3 コンテナ構成を詳しく解説（省略: 本文に挿入可）

---

## 環境変数（.env）設定

```
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=secret
DB_ROOT_PASSWORD=rootpassword
```

---

## コンテナの起動方法

```sh
docker compose up -d
```

---

## Laravel プロジェクト作成手順

### ① app コンテナに入る

```sh
docker exec -it laravel_app bash
```

### ② Laravel をインストール

```
composer create-project laravel/laravel .
```

このコマンドは以下を意味します:

- create-project → 新規プロジェクトを作成
- laravel/laravel → テンプレート
- `.` → 今いるディレクトリにインストール

---

## MySQL 接続方法（TablePlus）

### 設定値

| 項目 | 値 |
|------|------|
| Host | 127.0.0.1 |
| Port | 3306 |
| User | DB_USERNAME |
| Password | DB_PASSWORD |
| DB name | DB_DATABASE |
| SSL Mode | Prefered（基本不要） |

### エラー例：caching_sha2_password

MySQL 8 の認証方式。SSL がないと拒否されることがある。  
TablePlus は自動的に SSL なしでも対応可能。

---

## 便利コマンド集

### コンテナ一覧

```sh
docker ps
```

### 再起動

```sh
docker compose restart
```

### 削除

```sh
docker compose down
```

### ボリューム含めて削除

```sh
docker compose down -v
```

---

## トラブルシュート

### WARN: The "DB_PASSWORD" variable is not set.

→ `.env` が読み込めていない状態  
→ `.env` を作成し、`docker compose --env-file .env up -d` で解決

---

以上、図解＋解説＋手順すべてを統合した完全版です。
