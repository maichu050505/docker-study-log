Docker 学習ノート

# Docker の構造（アーキテクチャ）

1. Docker クライアント
   Docker を操作するには、クライアントから Docker が動作するサーバーに対してリクエストを送る。
   - docker コマンドの実行(CLI)
   - Docker Desktop という GUI ツール
2. Docker デーモン
   クライアントからリクエストを受け付け、コンテナの作成や実行、イメージのプルなどを管理するソフトウェア。
   Docker クライアントと Docker デーモン間は REST API を使って通信する。
3. レジストリ
   Docker Hub などのレジストリからイメージをプルする。

# Docker Desktop

Windows や Mac で Docker を使うためのソフトウェア。GUI ツール。
Docker クライアントや Docker デーモンなどが含まれている。
大企業が使う時は有料、個人利用・学習目的・中小企業の利用は無料。

## Apple シリコン Mac の場合

- イメージが Apple シリコン Mac で使えるか調べるには、Docker Hub の OS/ARCH に、「linux/arm64/v8」と書いてあれば、Apple シリコン Mac でも使える。
- Rosetta2 のインストールは昔は必要だったけど、2025 年の今はほぼ不要。
- Docker Desktop を公式からダウンロード（https://www.docker.com/products/docker-desktop/）
  Mac が M1/M2/M3 なら、Download for Mac - Apple Silicon をクリック。

# docker コマンド

docker 対象　操作

## イメージのプル

docker image pull イメージ名:タグ名

- タグを指定しないと、最新版のイメージがプルされる。

## コンテナの作成

docker container create

## コンテナの実行

docker container run

## Apache のコンテナ作成

- Apache とは、オープンソースの Web サーバーソフトウェア。
- コンテナを作るにはイメージが必要なので Docker Hub で登録されているイメージを使う。イメージの名前は、「httpd」
- ターミナルで、「docker container run --name apache01 -p 8080:80 -d httpd」を実行。
- 「Unable to find image 'httpd:latest' locally」
  → ローカルに httpd がないので Docker Hub からダウンロードするよ
- 「Pull complete
  Pull complete
  Pull complete
  ...
  Status: Downloaded newer image for httpd:latest」
  → httpd:latest のイメージを正常にダウンロード（pull）できました！
- 一番最後のこれ
  「e072f63fb5e88e2502a2f261e6486797f40edfcecf41f4f06380e76e0203075d」
  これは 起動したコンテナの ID！
- 「docker ps」で、現在のコンテナ一覧が見れる。
  STATUS: Up XXX seconds
  PORTS: 0.0.0.0:8080->80/tcp
  のように出ていれば OK
- http://localhost:8080/ で、Apache のテストページ (It works!)が出れば OK!
- Docker Desktop からも確認できる。実行中の状態なら緑色の丸で表示される。

## コンテナの停止

- 「docker container stop コンテナ名」
- docker container stop apache01
- http://localhost:8080/ で、「このサイトにアクセスできません」が出れば OK
- Docker Desktop でも、白い丸で表示される。

## コンテナの削除

- 「docker container rm コンテナ名」
- docker container rm apache01

## 複数のコンテナをラクに作る Docker Compose

- Docker Compose とは、一度に複数のコンテナを作成・実行できるソフトウェア。
- Docker Desktop にデフォルトで同梱されている。
- docker compose コマンドを使う。
- docker compose でコンテナを作成するには、YAML（ヤムル）形式のファイルでどういうコンテナを作成するかオプションをまとめて記述する。
- YAML とは、構造化データを記述できる記法。
  例）
  キー 1: 値 1
  キー 2:
  キー 2-1: 値 2-1
  キー 2-2: 値 2-2
- YAML ファイルのデフォルトの名前は、compose.yaml
- Apache コンテナを作る compose.yaml の例）
  ```yaml
  services:
   web:
   image: httpd:2.4
   container_name: apache01
   ports: - "8080:80"
  ```
- ステップ 1：
  作成したコンテナの情報を整理する。
- ステップ 2:
  Docker Compose ファイルを作成する。
  必要に応じて、Dockerfile というファイルも用意する。
- ステップ 3:
  コマンドを使って、コンテナを作成・実行する。
  ステップ 2 で作成したファイルをフォルダに配置し、そのパスの上で、「docker compose up -d」を実行。

## 実際に作ってみる。

- chap4/apache/compose.yaml を作成。（インデントは Tab キーではなく、半角スペースで）

```yaml
services:
  web:
    image: httpd:2.4
    ports:
      - "8080:80"
```

- Docker Desktop を立ち上げる。
- docker compose up -d を実行（-d はバックグラウンドで実行するオプション）
- 「Container コンテナ名 Started」というログが表示されれば成功！
- docker container ls で、実行中のコンテナを確認。
- docker container ls -a で、停止しているコンテナも含めて全てのコンテナを確認。
- STATUS に「Up 6 minutes」のように表示されていれば OK!
- コンテナの一覧ではなく、compose プロジェクトの一覧が見たい時は、docker compose ls
- http://localhost:8080/ でも確認できる。

## コンテナの停止

docker compose stop

## 作成済みのコンテナを起動する

docker compose start

- 複数のコンテナがある場合は、まとめて起動する。

## Docker Compose ファイルの書き方

- 複数の値を記述したい場合は、先頭に「-」をつけて、半角スペース。
- コメントは、先頭に「#」をつける。

```yaml
services: # コンテナの定義を書く
  web: # コンテナ名
    image: httpd:2.4
    ports:
      - "8080:80" # 「ホストのポート番号:コンテナのポート番号」
```

- sercives 以外にも、ネットワークに関する networks やデータ保存に関する volumes などの大項目がある。

```yaml
networks:
  netweb01:
volumes:
  volweb01:
```

## コンテナへファイルをコピーする

docker compose cp ホストのファイル名 コンテナ名: コンテナ内のファイルパス

- Docker ホストとは、Docker を動かしている PC のこと

## Docker ホストへファイルをコピー

docker compose cp コンテナ名: コンテナ内のファイルパス ホストのファイルパス

- 例 Apache コンテナの中にあるファイル「/usr/local/apache2/htdocs/index.html」を、Docker ホストのカレントディレクトリ(.)にコピーする。

docker compose cp web:/usr/local/apache2/htdocs/index.html .
を実行すると、Apache コンテナを作成すると自動でできる index.html を、Docker を動かしている PC の現在のディレクトリにコピーできる。
この時の「web」というのは、compose.yaml で、指定した、

```yaml
services: # コンテナの定義を書く
  web: # コンテナ名
    image: httpd:2.4
    ports:
      - "8080:80" # 「ホストのポート番号:コンテナのポート番号」
```

コンテナ名の web。
実際のコンテナ名は、apache-web-1 のようになる。プロジェクト名-web-1 のようになる。
docker が起動している状態で、「docker compose ps」すると、NAME のところに docker の名前が出てくる。

コピーした index.html を編集して、今度は、コンテナ内にコピーする。

docker compose cp index.html web:/usr/local/apache2/htdocs/index.html

## コンテナの削除（紐づくネットワークも削除）

docker compose down

- コンテナを実行中でも使用可能。

## コンテナの削除（紐づくネットワークは削除しない）

docker compose rm

- コンテナ実行中は使用不可。コンテナの停止と同時に行うには、docker compose rm -s

- 削除した後は、docker container ls や docker compose ls 　で削除したことを確認できる。

## イメージも合わせて削除する

docker compose down --rmi all

- 削除した後は、docker image ls で確認できる。Docker Desktop でも確認できる

# MariaDB コンテナの構築

- chap4/mariadb/compose.yaml を作成。

```yaml
services:
  db: # コンテナ名（サービス名）
    image: mariadb:10.7 # MariaDBイメージ
    environment: # 環境変数
      MARIADB_ROOT_PASSWORD: rootpass # ルートユーザーのパスワード
      MARIADB_DATABASE: testdb # データベース名
      MARIADB_USER: testuser # データベースのユーザー名
      MARIADB_PASSWORD: testpass # データベースのパスワード
    volumes: # データを永続化する設定
      - db-data:/var/lib/mysql
volumes: # データを永続化する設定
  db-data:
```

- compose.yaml があるディレクトリ(mariadb)に移動して、docker compose up -d
- Container 【コンテナ名】 Started が表示されれば成功！

- どんな環境変数が用意されているか調べるには、イメージのドキュメントを参照する。(Docker Hub の MariaDB のページ。)

## コンテナ内でコマンドを実行する

- 起動中のコンテナ内で、コマンドを実行する

docker compose exec コンテナ名 コンテナで実行したいコマンド

### コンテナにインストールされた MariaDB のバージョンを確認

docker compose exec db mariadb --version
db がコンテナ名（サービス名）。実際のコンテナ名は、フォルダ名-サービス名-1 なので、mariadb-db-1
mariadb がコマンド名。コンテナの中にインストールされているコマンドを呼んでいる。
--version が引数

### コンテナ内でシェルを立ち上げる

- シェルとは、受け取ったコマンドをカーネルに伝えるプログラム。
- カーネルとは、OS の心臓、PC 全体の管理者。
- コンテナ内でシェルを立ち上げ、そのシェルを介してコマンドを実行する = 「コンテナの中に入る」と表現される。
- コンテナ内で bash というシェル を立ち上げる
  docker compose exec コンテナ名 /bin/bash
  docker compose exec db /bin/bash を実行すると、
  root@95609e142d14:/# などと出てくる。ここで、
  mariadb --version を実行できる。
  次に、データベースのログインする。
  mariadb -u testuser -D testdb -p を実行（mariadb -u データベースのユーザー名 -D データベース名 -p）
  Enter password と出るので、パスワードを入力して Enter
  Welcome to the MariaDB monitor. 〜 省略 〜　と出たらデータベースに接続成功！
  MariaDB の CLI を終了（データベースをログアウト）するには、「¥q」（Mac では「\q」）\は、option + ¥
  立ち上げたシェルを終了するには、exit を入力してエンター。

# WordPress + MariaDB コンテナを構築する

- Docker Compose ファイルに、WordPress と MariaDB、2 つのコンテナの定義を書く。
- WordPress のイメージには、WordPress から接続するデータベースの情報を設定する、環境変数が用意されている。
- depends_on:を使って、まず MariaDB → その次に WordPress の順に作成する。
- WordPress の image のバージョンは、 image: wordpress:php8.2-apache が推奨。
- :latest は危険！不具合が起きた時に、実際にバージョンいくつかわからなくなる。
- chap4/wordpress/compose.yaml を作成

```yaml
services:
  db: # MariaDBのコンテナ
    image: mariadb:10.11
    environment:
      MARIADB_ROOT_PASSWORD: rootpass # ルートユーザーのパスワード
      MARIADB_DATABASE: wordpress # データベース名
      MARIADB_USER: wordpress # データベースのユーザー名
      MARIADB_PASSWORD: wordpress # データベースのパスワード
    volumes:
      - db-data:/var/lib/mysql
  wordpress:
    image: wordpress:php8.2-apache
    depends_on:
      - db
    environment:
      WORDPRESS_DB_HOST: db # 接続先(MariaDB)のコンテナ名
      WORDPRESS_DB_NAME: wordpress # 接続先データベース（MariaDB）のデータベース名
      WORDPRESS_DB_USER: wordpress # 接続先データベース（MariaDB）のユーザー名
      WORDPRESS_DB_PASSWORD: wordpress # 接続先データベース(MariaDB)のパスワード
    ports:
      - "8080:80"
    volumes:
      - wordpress-data:/var/www/html
volumes:
  db-data:
  wordpress-data:
```

- wordpress フォルダに移動して、docker compose up -d を実行してコンテナを作成。
- MariaDB と WordPress、それぞれ別のコンテナを作成している。
- http://localhost:8080/で確認。

# コンテナ内のデータを残す

- ボリューム： Docker が管理する記憶領域にデータを永続化する仕組み。
  データを変更する際は、データを直接操作するのではなく、コンテナを通して行う。
  データベースのデータを直接変更することのないデータに向いている。
  docker compose down でコンテナを削除する際、デフォルトでは、ボリュームは削除されない。再度コンテナを作成した際にそのボリュームをマウントすればデータを利用できる。
  Docker Compose ファイルに、
  ```yaml
  Volumes: ボリューム名
  ```
  で、ボリュームを作成できる。services の項目配下にも、volumes: という項目を作る。
  ```yaml
  services:
    コンテナ名:
    image: 使用するイメージ
    volumes:
      - 「volumes」に定義したボリューム名:コンテナ内のパス
  volumes: ボリューム名
  ```
- バインドマウント: ホストの OS のフォルダやファイルをマウントする仕組み。
  データを変更するときは、ホスト OS のファイルを直接変更することでコンテナ内にも自動で反映される。
  変更頻度の高いデータに向いている。
  ただし、ホストマシンによってフォルダ構成が異なる可能性が高いので、汎用性ではないのがデメリット。
  Docker Compose ファイルで、services の下に、volumes を記載。

  ```yaml
  services:
    コンテナ名:
      image: 使用するイメージ名
      volumes:
        - ホストOSのフォルダ:コンテナ内のパス
  ```

  ホスト OS のデータを削除すれば、コンテナ内のデータも合わせて削除される。
  公式は「本番運用ではボリュームを推奨」、開発ではバインドマウントも OK。

  chap4/wordpress02/compose.yaml を作成

  ```yaml
  services:
    db: # MariaDBのコンテナ
      image: mariadb:10.11
      environment:
        MARIADB_ROOT_PASSWORD: rootpass # ルートユーザーのパスワード
        MARIADB_DATABASE: wordpress # データベース名
        MARIADB_USER: wordpress # データベースのユーザー名
        MARIADB_PASSWORD: wordpress # データベースのパスワード
      volumes:
        - db-data:/var/lib/mysql
  wordpress:
    image: wordpress:php8.2-apache
    depends_on:
      - db
    environment:
      WORDPRESS_DB_HOST: db # 接続先(MariaDB)のコンテナ名
      WORDPRESS_DB_NAME: wordpress # 接続先データベース（MariaDB）のデータベース名
      WORDPRESS_DB_USER: wordpress # 接続先データベース（MariaDB）のユーザー名
      WORDPRESS_DB_PASSWORD: wordpress # 接続先データベース(MariaDB)のパスワード
    ports:
      - "8080:80"
    volumes:
      - ./html:/var/www/html # バインドマウントに変更 ./htmlは、compose.ymlのあるフォルダ内のhtmlフォルダ。
  volumes:
    db-data: # ボリュームからwordpress-data:を削除
  ```

docker compose up -d でコンテナを作成すると、wordpress02 フォルダ内に html フォルダができ、その中に wordpress ファイルが入っている。

- 実務で、WordPress/PHP のバージョンアップの際にテスト環境を作る場合は、MariaDB = ボリューム、WordPress = バインドマウント がベスト！
  WordPress ファイルは、vs code で編集できた方が便利。

# コンテナのネットワーク

- WordPress コンテナは、Web ブラウザからアクセスできたり、MariaDB コンテナにデータを保存する。これがコンテナ外・コンテナ間のネットワーク通信。
- Docker では 3 つのネットワークがデフォルトで作られる。
- Docker のネットワーク一覧を表示
  Docker network ls 　（どの階層で実行しても OK）
  NAME 欄に、bridge, host, null がある。
  1. bridge: コンテナ間、コンテナ外と通信できるネットワーク。主に使われる。
  2. host: Docker ホストのネットワークをそのまま使う。
  3. none: コンテナ間、コンテナ外とも通信できない。
- Docker Compose では、Docker Compose で定義したコンテナが通信できる専用のネットワークがデフォルトで作成される。
- compose.yml を書いて up するだけで、ユーザー定義 bridge ネットワークが自動で作られる
- link オプションは古い機能で非推奨。

# すぐに使えるDocker設定ファイル（MariaDB + phpMyAdmin）
- phpMyAdminとは、MariaDBを操作できるGUIツール。
- chap5/phpmyadmin/compose.yaml を作成
```yaml
services:
  db: # MariaDBのコンテナ
    image: mariadb:10.11
    environment:
      MARIADB_ROOT_PASSWORD: rootpass # ルートユーザーのパスワード
      MARIADB_DATABASE: testdb # データベース名
      MARIADB_USER: testuser # データベースのユーザー名
      MARIADB_PASSWORD: testpass # データベースのパスワード
    volumes:
      - db-data:/var/lib/mysql
  phpmyadmin:
   image: phpmyadmin:5.2.3
   depends_on:
    - db
   environment:
    PMA_HOST: db # MariaDBのコンテナ名を設定
    PMA_USER: testuser
    PMA_PASSWORD: testpass
   ports:
    - "8080:80"
   volumes:
    - phpmyadmin-data:/sessions
volumes:
 db-data:
 phpmyadmin-data:
```
- Docker Desktopを起動した状態で、phpmyadminフォルダで、docker compose up -d を実行
- http://localhost:8080/ を開くと、phpMyAdminが見れる。