
# Colima + Docker + Laravel 開発環境構築手順書

## 目次
1. [概要](#概要)
2. [必要ツールのインストール](#必要ツールのインストール)
3. [Colima のセットアップ](#colima-のセットアップ)
4. [Docker プロジェクト構成](#docker-プロジェクト構成)
5. [Dockerfile の説明](#dockerfile-の説明)
6. [docker-compose.yml の説明](#docker-composeyml-の説明)
7. [環境構築手順](#環境構築手順)
8. [図解（構成イメージ）](#図解構成イメージ)

---

## 概要
本書は **Colima + Docker + Laravel** を用いてローカル開発環境を構築するための手順書です。

---

## 必要ツールのインストール
### Homebrew
macOS のパッケージマネージャー。

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Docker
```bash
brew install docker docker-compose
```

### Colima
```bash
brew install colima
```

---

## Colima のセットアップ
```bash
colima start --cpu 2 --memory 4 --disk 60 --vm-type vz
```

---

## Docker プロジェクト構成

```
laravel_docker/
├── docker/
│   ├── php/
│   │   └── Dockerfile
│   └── nginx/
│       └── default.conf
├── docker-compose.yml
└── .env
```

---

## Dockerfile の説明
PHP（FPM版）のコンテナをビルドする設定。

（※詳細は前回の内容に準拠）

---

## docker-compose.yml の説明

- app: PHP-FPM（Laravel本体を動かす）
- web: Nginx（LaravelのWebサーバ）
- db: MySQL（データベース）

---

## 環境構築手順

### 1. プロジェクト作成
```bash
mkdir laravel_docker
cd laravel_docker
```

### 2. コンテナ起動
```bash
docker compose up -d
```

### 3. Laravel インストール
```bash
docker exec -it laravel_app bash
composer create-project laravel/laravel .
```

---

## 図解（構成イメージ）

```
         ┌──────────────────────┐
         │        Colima        │
         │   (仮想Linux環境)    │
         └─────────┬────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
 ┌──────▼───────┐     ┌──────▼───────┐
 │   Docker      │     │   Docker     │
 │   Container   │     │   Container  │
 │   (Nginx)     │     │   (MySQL)    │
 └──────┬────────┘     └──────┬───────┘
        │                      │
 ┌──────▼────────┐
 │   Docker       │
 │   Container    │
 │ (PHP-FPM App)  │
 └────────────────┘
```

---

以上が図解付きの手順書です。
