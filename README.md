# 手順書

## vim

### Amazon Linux 2 でのvimのインストール方法
```
sudo yum install vim -y
```

## Docker

### Docker インストール方法 & 自動起動化
```
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
```
dockerをインストールをすることができましたが、dockerは全てのユーザーが使えるようになってはいません。

デフォルトのユーザー（ec2-user）でもsudoつけずにdockerコマンドを立たけるように、dockerグループに追加します。
```
sudo usermod -a -G docker ec2-user
```
usermodを反映するために一度ログアウトする必要があります。

sshの場合は一度ログアウトしログインしなおすことで反映させることができます。

screenのウィンドウ等もすべて一度終了させる(**exit**コマンドで)とよいでしょう。

## Docker Compose

### Docker Compose インストール方法
```
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
mkdir -p $DOCKER_CONFIG/cli-plugins
curl -SL https://github.com/docker/compose/releases/download/v5.1.2/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
```
インストールできたかどうかの確認
```
docker compose version
```
## 最新版のbuildxのインストール
```
mkdir -p ~/.docker/cli-plugins
ARCH=$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/')
BUILDX_URL=$(curl -s https://api.github.com/repos/docker/buildx/releases/latest | grep "browser_download_url.*linux-$ARCH" | cut -d '"' -f 4)
curl -L $BUILDX_URL -o ~/.docker/cli-plugins/docker-buildx
chmod +x ~/.docker/cli-plugins/docker-buildx
```
**docker buildx version** で確認して、 **v0.34.1** (またはそれ以上のバージョン) が表示されたら大丈夫です！

## Dockerfileの作成
まずは作業用のディレクトリを作ってその中に移動しましょう。
```
mkdir dockertest
cd dockertest
```
次はカレントディレクトリに Dockerfile を作成しましょう。
```
vim Dockerfile
```
中身はこちら
https://github.com/Tatsuo1958/kadai/blob/main/Dockerfile
compose.yml を作成しましょう。
```
vim compose.yml
```
中身はこちら
https://github.com/Tatsuo1958/kadai/blob/main/compose.yml

次に、webコンテナ内の **/var/www/upload/image/** を、nginxが、インターネットに対して **/image/** として配信するように、nginxの設定ファイル **nginx/conf.d/default.conf** を作成しましょう。
まず設定ファイル用のディレクトリを作ります。
```
mkdir nginx
mkdir nginx/conf.d
```
設定ファイルを作成
```
vim nginx/conf.d/default.conf
```
中身はこちら
https://github.com/Tatsuo1958/kadai/blob/main/nginx/conf.d/default.conf

そして **docker compose build** をし、 **docker compose up** を行います。
```
docker compose build
docker compose up
```

## 画像アップロードのアプリケーション実装
データベースの掲示板用テーブルに画像を保存するパスを入れておくカラムを追加しましょう。

MySQLクライアントを起動
```
docker compose exec mysql mysql example_db
```
MySQLクライアントを起動したら以下のSQLを実行し，テーブルを作成します。
```
CREATE TABLE `bbs_entries` (
    `id` INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `body` TEXT NOT NULL,
    `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP,
    `image_filename` TEXT DEFAULT NULL
);
```
最後にPHPのプログラムを実装します。
ファイルを置くディレクトリを作成
```
mkdir public
```
仮に、 **public/bbsimagetest.php** として作成します。
```
sudo vim public/bbsimagetest.php
```
中身はこちら
https://github.com/Tatsuo1958/2026-/blob/main/dockertest/public/bbsimagetest.php

最後にDockerを起動します
```
docker compose up
```

**http://EC2インスタンスのIPアドレス/bbsimagetest.php** にブラウザからアクセスして動作を確認してみましょう。

