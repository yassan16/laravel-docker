# PHP公式のDockerイメージ（FPM版）
#   FPM (FastCGI Process Manager) は、Nginxなどと連携して動く高速なPHP実行環境
FROM php:8.4-fpm

# 必要パッケージとよく使う拡張
# 1. apt-get update: パッケージリストを更新。
#
# 2. apt-get install -y --no-install-recommends: 最小限の依存関係で、以下のパッケージをインストール
#   - git：Composerなどで依存パッケージを取る際に必要。
#   - unzip：ZIP展開が必要なComposerパッケージに使われる。
#   - libzip-dev：PHPの zip 拡張をビルドするために必要。
#   - libicu-dev：intl（国際化）拡張をビルドするために必要。
#
# 3. docker-php-ext-install: PHP公式イメージが提供するビルドスクリプト。
#   pdo_mysql zip intl opcache の4つの拡張をインストール。
#   - pdo_mysql：MySQL接続用（LaravelのDB接続に必須）
#   - zip：ZIP処理用（Composerなどで使用）
#   - intl：日付/ロケール関連処理に必要
#   - opcache：パフォーマンス向上（キャッシュ）
#
# 4. rm -rf /var/lib/apt/lists/*: イメージサイズ削減のため、キャッシュを削除。
RUN apt-get update && apt-get install -y --no-install-recommends \
    git unzip libzip-dev libicu-dev \
  && docker-php-ext-install -j$(nproc) pdo_mysql zip intl opcache \
  && rm -rf /var/lib/apt/lists/*

# Composer を公式イメージからコピー
# 1. multi-stage buildの手法。
#   Composer専用イメージから、バイナリ(/composer)だけをコピー。
#   → Composerのセットアップを最小限で済ませる（早くて軽い）
#
# 2. COMPOSER_ALLOW_SUPERUSER=1
#   コンテナ内ではroot権限で動作するため、Composerが警告を出さないように設定。
COPY --from=composer/composer:latest-bin /composer /usr/local/bin/composer
ENV COMPOSER_ALLOW_SUPERUSER=1

# コンテナ内で作業を行うディレクトリ。
#   Laravelや一般的なPHPアプリのルートパスに合わせて /var/www/html に設定。
WORKDIR /var/www/html

# 依存だけ先に入れてキャッシュを効かせる
# 1. COPY composer.json composer.lock ./
#   依存情報だけを先にコピーすることで、変更がなければキャッシュが効く。
#
# 2. composer install
#   依存ライブラリを vendor にインストール。
#  - --no-dev: 開発用パッケージを除外（本番向け）
#  - --prefer-dist: Gitではなくzipで取得（高速・軽量）
#  - --no-interaction: 対話なし
#  - --optimize-autoloader: オートローダを最適化して起動を高速化
# COPY composer.json composer.lock ./
# RUN composer install --no-dev --prefer-dist --no-interaction --optimize-autoloader
# -> Laravelは後で作成するため、COPYやcomposer installは不要になった

# プロジェクト全体を /var/www/html にコピー。
#   これを最後に行うことで、アプリの変更があってもComposerインストールのキャッシュが無駄にならない。
COPY . .

# PHP-FPM はベースの CMD で起動