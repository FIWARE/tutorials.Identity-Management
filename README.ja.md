[![FIWARE Banner](https://fiware.github.io/tutorials.Identity-Management/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Security](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/security.svg)](https://github.com/FIWARE/catalogue/blob/master/security/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Identity-Management.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

<!-- prettier-ignore -->

このチュートリアルは
、[FIWARE Keyrock](https://fiware-idm.readthedocs.io/en/latest/) の紹介です。こ
れは、FIWARE サービスに **ID 管理** (Identity Management) を導入する Generic
Enabler です。このチュートリアルでは、ユーザと組織の作成方法について、後のチュー
トリアルで、それらにロールと権限の割り当てる方法について説明します。

このチュートリアルでは、**Keyrock** GUI を使用したインタラクションの例や
、**Keyrock** REST API へのアクセスに使用される [cUrl](https://ec.haxx.se/) コマ
ンド
、[Postman のドキュメント](https://fiware.github.io/tutorials.Identity-Management/)も
使用できます。

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/5f9e1736f979b86ec94a)
[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/FIWARE/tutorials.Identity-Management/tree/NGSI-v2)

## コンテンツ

<details>
<summary>詳細 <b>(クリックして拡大)</b></summary>

-   [ID 管理](#identity-management)
    -   [ID 管理の標準概念](#standard-concepts-of-identity-management)
    -   [:arrow_forward: ビデオ : Keyrock のイントロダクション](#arrow_forward-video--introduction-to-keyrock)
-   [前提条件](#prerequisites)
    -   [Docker](#docker)
    -   [WSL](#wsl)
-   [アーキテクチャ](#architecture)
    -   [Keyrock の設定](#keyrock-configuration)
    -   [MySQL の設定](#mysql-configuration)
-   [起動](#start-up)
    -   [Keyrock MySQL データベースからの直接読み込み](#reading-directly-from-the-keyrock-mysql-database)
    -   [Keyrock 内の UUIDs](#uuids-within-keyrock)
    -   [:arrow_forward: ビデオ: Keyrock GUI でのユーザ・アカウントの作成](#arrow_forward-video--creating-user-accounts-with-the-keyrock-gui)
    -   [ログイン](#logging-in)
        -   [パスワードでトークンを作成](#create-token-with-password)
        -   [トークンを介してユーザ情報を取得](#get-user-information-via-a-token)
        -   [トークンをリフレッシュ](#refresh-token)
-   [ユーザ・アカウントの管理](#administrating-user-accounts)
    -   [登場人物 (Dramatis Personae)](#dramatis-personae)
    -   [ユーザ CRUD アクション](#user-crud-actions)
        -   [ユーザを作成](#creating-users)
        -   [ユーザ情報を取得](#read-information-about-a-user)
        -   [すべてのユーザの一覧を取得](#list-all-users)
        -   [ユーザを更新](#update-a-user)
        -   [ユーザを削除](#delete-a-user)
-   [組織下でのユーザ・アカウントのグルーピング](#grouping-user-accounts-under-organizations)
    -   [組織 CRUD アクション](#organization-crud-actions)
        -   [組織の作成](#create-an-organization)
        -   [組織の詳細を取得](#read-organization-details)
        -   [すべての組織の一覧を取得](#list-all-organizations)
        -   [組織を更新](#update-an-organization)
        -   [組織を削除](#delete-an-organization)
    -   [組織内のユーザ](#users-within-an-organization)
        -   [組織のメンバとしてユーザを追加](#add-a-user-as-a-member-of-an-organization)
        -   [組織のオーナーとしてユーザを追加](#add-a-user-as-an-owner-of-an-organization)
        -   [組織内のユーザの一覧を取得](#list-users-within-an-organization)
        -   [組織内のユーザ・ロールを取得](#read-user-roles-within-an-organization)
        -   [組織からユーザを削除](#remove-a-user-from-an-organization)
-   [次のステップ](#next-steps)

</details>

<a name="identity-management"></a>

# ID 管理

> "If one meets a powerful person — ask them five questions: ‘What power have
> you got? Where did you get it from? In whose interests do you exercise it? To
> whom are you accountable? And how can we get rid of you?’"
>
> — Anthony Wedgwood Benn (The Five Essential Questions of Democracy)

コンピュータ・セキュリティ用語では、ID 管理 (Identity management) は、"適切な人
が適切なタイミングで適切なリソースに適切な理由でアクセスできるようにするセキュリ
ティおよびビジネス規律"です。さまざまなシステム間のリソースへの適切なアクセスを
確保する必要性に対処します。

FIWARE フレームワークは一連の独立したコンポーネントで構成されており、セキュリテ
ィのチャプターでは、システム内のどのリソースに誰がアクセスするのかについて、これ
らのコンポーネントの共通のニーズを実装することを目指しています。リクエストを行う
人またはサービスの身元を知る必要があります。FIWARE **Keyrock** Generic Enabler
は、ID 管理システムの共通特性のすべてをすぐに設定し、他のコンポーネントが標準の
認証メカニズムを使用して、業界標準のプロトコルに基づいてリクエストを受け入れるか
拒否することができるようにします。

したがって、ID 管理は、システム内で ID を得る方法、ID を保護する方法、およびパス
ワードやネットワーク・プロトコルなどの周囲の技術を取り上げる問題を扱います。

<a name="standard-concepts-of-identity-management"></a>

## ID 管理の標準概念

**Keyrock** ID 管理データベースには、次の共通オブジェクトがあります :

-   **User** - 電子メールとパスワードを使用して自分自身を識別できる、登録済みの
    ユーザ。ユーザには、個別にまたはグループとして権利を割り当てることができます
-   **Application** - 一連のマイクロ・サービスで構成された任意のセキュアな
    FIWARE アプリケーション
-   **Organization** - 一連の権利を割り当てることができるユーザのグループ。組織
    の権利を変更すると、その組織のすべてのユーザのアクセスが影響を受けます
-   **OrganizationRole** - ユーザは組織のメンバまたは管理者になることができます
    。管理者は組織にユーザを追加または削除できます。メンバは組織のロールと権限を
    取得するだけです。これにより、各組織はメンバに対して責任を持つことができ、ス
    ーパー管理者 (super-admin) がすべての権限を管理する必要がなくなります
-   **Role** - ロールは、一連のアクセス許可の説明的なバケットです。ロールは、単
    一のユーザまたは組織に割り当てることができます。サインインしたユーザは、自分
    のすべてのロールとその組織に関連付けられているすべてのロールのすべての権限を
    取得します
-   **Permission** - システム内のリソース上で何かを行う能力

さらに、FIWARE アプリケーション内で、2 つの人以外のアプリケーション (non-human
application) のオブジェクトを保護することができます。

-   **IoTAgent** - IoT センサとコンテキスト・ブローカー間のプロキシ
-   **PEPProxy** - ユーザの権利を確認する Generic Enabler 間での使用のためのミド
    ルウェア

オブジェクト間の関係は以下のようになります :

![](https://fiware.github.io/tutorials.Identity-Management/img/entities.png)

<a name="arrow_forward-video--introduction-to-keyrock"></a>

## :arrow_forward: ビデオ : Keyrock のイントロダクション

[![](https://fiware.github.io/tutorials.Step-by-Step/img/video-logo.png)](https://www.youtube.com/watch?v=dHyVTan6bUY "Introduction")

イントロダクションのビデオを見るには上記の画像をクリックしてください。

<a name="prerequisites"></a>

# 前提条件

<a name="docker"></a>

## Docker

物事を単純にするために、両方のコンポーネントが [Docker](https://www.docker.com)
を使用して実行されます。**Docker** は、さまざまコンポーネントをそれぞれの環境に
分離することを可能にするコンテナ・テクノロジです。

-   Docker Windows にインストールするには
    、[こちら](https://docs.docker.com/docker-for-windows/)の手順に従ってくださ
    い
-   Docker Mac にインストールするには
    、[こちら](https://docs.docker.com/docker-for-mac/)の手順に従ってください
-   Docker Linux にインストールするには
    、[こちら](https://docs.docker.com/install/)の手順に従ってください

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義して実行する
ためのツールです
。[YAML file](https://raw.githubusercontent.com/Fiware/tutorials.tutorials.Identity-Management/master/docker-compose.yml)
ファイルは、アプリケーションのために必要なサービスを構成するために使用します。つ
まり、すべてのコンテナ・サービスは 1 つのコマンドで呼び出すことができます
。Docker Compose は、デフォルトで Docker for Windows と Docker for Mac の一部と
してインストールされますが、Linux ユーザ
は[ここ](https://docs.docker.com/compose/install/)に記載されている手順に従う必要
があります。

## WSL

シンプルな bash スクリプトを使用してサービスを開始します。Windows ユーザは
[を使用して Windows に Linux をインストールする方法](https://learn.microsoft.com/ja-jp/windows/wsl/install) をダウンロードして、Windows 上の Linux ディスト
リビューションと同様のコマンドライン機能を提供する必要があります。

<a name="architecture"></a>

# アーキテクチャ

このイントロダクションでは
、[Keyrock](https://fiware-idm.readthedocs.io/en/latest/) Identity Management
Generic Enabler という 1 つの FIWARE コンポーネントのみを使用します。**Keyrock**
単独での使用は、アプリケーションが _“Powered by FIWARE”_ と認定するには不十分で
す。さらに、**MySQL** データベースにユーザ・データを保存する予定です。

全体的なアーキテクチャは、次の要素で構成されます :

-   1 つの **FIWARE Generic Enabler** :

    -   FIWARE [Keyrock](https://fiware-idm.readthedocs.io/en/latest/) は補完的
        な ID 管理システムを提供します :
        -   アプリケーションとユーザのための認証システム
        -   ID 管理のアドミニストレーションのための Web サイトのグラフィカル・フ
            ロントエンド
        -   HTTP リクエストによる ID 管理用の同等の REST API

-   1 つの [MySQL](https://www.mysql.com/) データベース :
    -   ユーザ ID、アプリケーション、ロール、およびパーミッションを保持するため
        に使用します

要素間のすべてのインタラクションは HTTP リクエストによって開始されるため、エンテ
ィティはコンテナ化され、公開されたポートから実行されます。

![](https://fiware.github.io/tutorials.Identity-Management/img/architecture.png)

チュートリアルの各セクションの具体的なアーキテクチャについては、以下で説明します
。

<a name="keyrock-configuration"></a>

## Keyrock の設定

```yaml
keyrock:
    image: quay.io/fiware/idm
    container_name: fiware-keyrock
    hostname: keyrock
    depends_on:
        - mysql-db
    ports:
        - "3005:3005"
        - "${KEYROCK_HTTPS_PORT}:${KEYROCK_HTTPS_PORT}" # localhost:3443
    environment:
        - IDM_DB_HOST=mysql-db
        - IDM_DB_PASS_FILE=/run/secrets/my_secret_data
        - IDM_DB_USER=root
        - IDM_HOST=http://localhost:3005
        - IDM_PORT=3005
        - IDM_HTTPS_ENABLED=true
        - IDM_HTTPS_PORT=${KEYROCK_HTTPS_PORT}
        - IDM_ADMIN_USER=admin
        - IDM_ADMIN_EMAIL=admin@test.com
        - IDM_ADMIN_PASS=1234
    secrets:
        - my_secret_data
```

`keyrock` コンテナは、2 つのポートでリッスンしている、Web アプリケーション・サー
バです :

-   Port `3005` は HTTP トラフィックで公開されているため、Web ページを表示して
    REST API とやりとりすることができます
-   Port `3443` は Web サイトおよび REST API の HTTPS トラフィックを保護するため
    に公開されています

> :information_source: **注** すべてのセキュアなアプリケーションで HTTPS を使用
> する必要がありますが、これを正しく行うには ** Keyrock** には信頼できる SSL 証
> 明書が必要です。デフォルトの証明書は自己認証されており、テスト目的で利用できま
> す。 証明書は、`/opt/fiware-idm/certs` の下にあるファイルを置き換えるためにボ
> リュームを付加することで上書きすることができます。
>
> 実稼働環境では、プレーンテキストを使用して機密情報を送信しないように、HTTPS 経
> 由ですべてのアクセスを行う必要があります。 また、設定された HTTPS リバース・プ
> ロキシの背後にあるプライベート・ネットワーク内で HTTP を使用することもできます
> 。
>
> HTTP プロトコルを提供するポート `3005` は、デモンストレーションの目的でのみ公
> 開されており、このチュートリアルでのインタラクションを簡素化するために、ポート
> 3443 で HTTPS を使用することもあります。
>
> Postman を使用しているときに HTTPS を使用して REST API にアクセスする場合は
> 、SSL 証明書の検証がオフであることを確認してください。HTTPS を使用して Web フ
> ロントエンドにアクセスする場合は、発行されたセキュリティ警告を受け入れてくださ
> い。

`keyrock` コンテナは、次に示す環境変数によってドライブされます :

| キー              | 値                      | 説明                                                                                                                                                     |
| ----------------- | ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IDM_DB_PASS       | `idm`                   | 接続する MySQL データベースのパスワード。**Docker Secrets** によって保護されています。下記を参照してください                                             |
| IDM_DB_USER       | `root`                  | デフォルトの MySQL ユーザのユーザ名。プレーン・テキストです                                                                                              |
| IDM_HOST          | `http://localhost:3005` | **Keyrock** アプリケーション・サーバのホスト名。ユーザ登録時にアクティベーション e-mail で使用されます                                                   |
| IDM_PORT          | `3005`                  | **Keyrock** アプリケーション・サーバで使用される HTTP トラフィックのためのポート。これは、衝突を避けるためにデフォルトの `3000` ポートから変更しています |
| IDM_HTTPS_ENABLED | `true`                  | HTTPS サポートを提供するかどうか。これは、オーバーライドされない限り、自己署名証明書を使用します                                                         |
| IDM_HTTPS_PORT    | `3443`                  | HTTP トラフィック用の **Keyrock** アプリケーション・サーバで使用されるポート。デフォルトの 443 から変更されています。                                    |

> :information_source: **注** この例では、**Docker Secrets** を使用して MySQL パ
> スワードを保護していることに注意してください。`_FILE` サフィックスを持つ
> `IDM_DB_PASS` を使用し、シークレット・ファイルの場所を参照します。これによりプ
> レーン・テキストの `ENV` 変数として、パスワードを公開することを避けることがで
> きます。`Dockerfile` イメージか `docker inspect` を使って読むことができる注入
> 変数 (an injected variable ) です。

> 次の変数のリストは、プロダクション・システムで `_FILE` サフィックスを持つシー
> クレットを使用して設定する必要があります :
>
> -   `IDM_SESSION_SECRET`
> -   `IDM_ENCRYPTION_KEY`
> -   `IDM_DB_PASS`
> -   `IDM_DB_USER`
> -   `IDM_ADMIN_ID`
> -   `IDM_ADMIN_USER`
> -   `IDM_ADMIN_EMAIL`
> -   `IDM_ADMIN_PASS`
> -   `IDM_EX_AUTH_DB_USER`
> -   `IDM_EX_AUTH_DB_PASS`

<a name="mysql-configuration"></a>

## MySQL の設定

```yaml
mysql-db:
    image: mysql:5.7
    hostname: mysql-db
    container_name: db-mysql
    expose:
        - "3306"
    ports:
        - "3306:3306"
    networks:
        - default:
    environment:
        - "MYSQL_ROOT_PASSWORD_FILE=/run/secrets/my_secret_data"
        - "MYSQL_ROOT_HOST=172.18.1.5"
    volumes:
        - mysql-db:/var/lib/mysql
    secrets:
        - my_secret_data
```

`mysql-db` コンテナは、単一ポートで待機しています :

-   Port `3306` は MySQL サーバのデフォルト・ポートです。これは公開されているの
    で、必要に応じて他のデータベース・ツールを実行してデータを表示することもでき
    ます

`mysql-db` コンテナは、次に示すような環境変数によってドライブされます :

| キー                | 値     | 説明                                                                                                                                                                                       |
| ------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| MYSQL_ROOT_PASSWORD | `123`  | **Docker Secrets** によって保護されている MySQL の `root` アカウントに設定されているパスワードを指定します。下記を参照してください                                                         |
| MYSQL_ROOT_HOST     | `root` | デフォルトでは、MySQL は `root'@'localhost` アカウントを作成します。このアカウントはコンテナ内からのみ接続できます。この環境変数を設定すると、他のホストからのルート接続が可能になります。 |

<a name="start-up"></a>

# 起動

インストールを開始するには、次の手順を実行します :

```console
git clone https://github.com/FIWARE/tutorials.Identity-Management.git
cd tutorials.Identity-Management
git checkout NGSI-v2

./services create
```

> **注** Docker イメージの最初の作成には最大 3 分かかります

その後、リポジトリ内で提供される
[services](https://github.com/FIWARE/tutorials.Identity-Management/blob/NGSI-v2/services)
Bash スクリプトを実行することによって、コマンドラインからすべてのサービスを初期
化することができます :

```console
./services <command>
```

ここで、`<command>` は、私たちがアクティベートしたいエクササイズに応じてかわりま
す。

> :information_source: **注:** クリーンアップをやり直したい場合は、次のコマンド
> を使用して再起動することができます :
>
> ```console
> ./services stop
> ```

<a name="reading-directly-from-the-keyrock-mysql-database"></a>

### Keyrock MySQL データベースからの直接読み込み

すべての ID 管理のレコードと関連性は、MySQL データベース内に保持されます。以下の
ように実行中の Docker コンテナを入力するとアクセスできます :

```console
docker exec -it db-mysql bash
```

```console
mysql -u <user> -p<password> idm
```

ここで、`<user>` と `<password>` は、`docker-compose` ファイル内で定義された値の
`MYSQL_ROOT_PASSWORD` と `MYSQL_ROOT_USER` に一致します。チュートリアルのデフォ
ルト値は、通常`root`、および `secret` です。

コマンドラインから SQL コマンドを入力することができます。例えば :

```SQL
select id, username, email, password from user;
```

**Keyrock** MySQL データベースは、ユーザ、パスワードなどの格納を含むアプリケーシ
ョン・セキュリティのあらゆる側面を扱います。アクセス権を定義し、OAuth2 認証プロ
トコルを扱います。完全なデータベース関係図
は[ここ](https://fiware.github.io/tutorials.Identity-Management/img/keyrock-db.png)に
あります。

<a name="uuids-within-keyrock"></a>

### Keyrock 内の UUIDs

**Keyrock** 内のすべての IDs とトークンは変更される可能性があります。レコードを
クエリするときは、以下の値を修正する必要があります。レコード IDs は Universally
Unique Identifiers - UUIDs を使用します。

| キー                   | 説明                                                                                                                                | サンプル値                                         |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| `keyrock`              | **Keyrock** サービスの場所の URL                                                                                                    | HTTP 用 `localhost:3005`, HTTPS 用`localhost:3443` |
| `X-Auth-token`         | ユーザとしてログインするときにヘッダで受け取ったトークン。言い換えれば、_"私は誰ですか？"_                                          | `51f2e380-c959-4dee-a0af-380f730137c3`             |
| `X-Subject-token`      | *"誰に問い合わせたいですか?"*を定義するリクエストに追加されたトークン。これは上記で定義した `X-Auth-token` を繰り返すこともできます | `51f2e380-c959-4dee-a0af-380f730137c3`             |
| `user-id`              | `user` テーブルで見つかった既存ユーザの id                                                                                          | `96154659-cb3b-4d2d-afef-18d6aec0518e`             |
| `organization-id`      | `organization` テーブルで見つかった、既存組織の id                                                                                  | `e424ed98-c966-46e3-b161-a165fd31bc01`             |
| `organization-role-id` | `owner` または `member` のいずれかの組織内でユーザが持つロールのタイプ                                                              | `member`                                           |

トークンは、一定期間後に期限切れになるように設計されています。使用している
`X-Auth-token` 値の有効期限が切れている場合は、再度ログインして新しいトークンを
取得してください。

<a name="arrow_forward-video--creating-user-accounts-with-the-keyrock-gui"></a>

## :arrow_forward: ビデオ: Keyrock GUI でのユーザ・アカウントの作成

[![](https://fiware.github.io/tutorials.Step-by-Step/img/video-logo.png)](https://www.youtube.com/watch?v=dtKsjGbJ7Xc " Creating User Accounts")

上の画像をクリックすると、**Keyrock** GUI でユーザ・アカウントを作成する方法を示
すビデオが表示されます

<a name="logging-in"></a>

## ログイン

ログイン画面では、既存ユーザが自分自身を識別し、その後の操作のためにトークンを取
得することができます。これは、**Keyrock** GUI の初期起動画面です :
`http://localhost:3005/idm` (または `https://localhost:3443/idm` と警告を受け入
れます)

![](https://fiware.github.io/tutorials.Identity-Management/img/log-in.png)

**Keyrock** アプリケーションに入るには、ユーザ名とパスワードを入力します。デフォ
ルトのスーパー管理ユーザ (super-admin user) は `admin@test.com` と `1234` の値を
持っています。

<a name="create-token-with-password"></a>

### パスワードでトークンを作成

次の例では、スーパー管理ユーザ を使用してログインします。GUI のログイン画面を使
用するのと同じです。URL `https://localhost:3443/v1/auth/tokens` はセキュアなシス
テムでも動作するはずです。

#### 1️⃣ リクエスト :

```console
curl -iX POST \
  'http://localhost:3005/v1/auth/tokens' \
  -H 'Content-Type: application/json' \
  -d '{
  "name": "admin@test.com",
  "password": "1234"
}'
```

#### レスポンス :

レスポンス・ヘッダは、誰がアプリケーションにログオンしているかを識別する
`X-Subject-token` を返します。このトークンは、以降のすべてのリクエストにアクセス
するために必要です。

```
HTTP/1.1 201 Created
X-Subject-Token: d848eb12-889f-433b-9811-6a4fbf0b86ca
Content-Type: application/json; charset=utf-8
Content-Length: 138
ETag: W/"8a-TVwlWNKBsa7cskJw55uE/wZl6L8"
Date: Mon, 30 Jul 2018 12:07:54 GMT
Connection: keep-alive
```

```json
{
    "token": {
        "methods": ["password"],
        "expires_at": "2018-07-30T13:02:37.116Z"
    },
    "idm_authorization_config": {
        "level": "basic",
        "authzforce": false
    }
}
```

<a name="get-user-information-via-a-token"></a>

### トークンを介してユーザ情報を取得

ユーザがログインすると、時間制限されたトークンがあれば、ユーザに関する詳細情報を
見つけることができます。

`{{X-Auth-token}}` と `{{X-Subject-token}}` は、以前のリクエストから取得しなけれ
ばなりません。上記のレスポンスの場合には、両方の変数を
`d848eb12-889f-433b-9811-6a4fbf0b86ca` に設定する必要があります
。`{{X-Auth-token}}` トークンで許可されたユーザが `{{X-Subject-token}}` トークン
を保持しているユーザについて問い合わせていることを示しています。この場合
、**Keyrock** アプリケーション内には 1 人のユーザしかいません。 そのユーザは自分
自身について質問しています。

#### 2️⃣ リクエスト :

```console
curl -X GET \
  'http://localhost:3005/v1/auth/tokens' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -H 'X-Subject-token: {{X-Subject-token}}'
```

#### レスポンス :

レスポンスは関連するユーザの詳細を返します :

```json
{
    "access_token": "51f2e380-c959-4dee-a0af-380f730137c3",
    "expires": "2018-07-30T13:02:37.000Z",
    "valid": true,
    "User": {
        "id": "admin",
        "username": "admin",
        "email": "admin@test.com",
        "date_password": "2018-07-30T09:55:38.000Z",
        "enabled": true,
        "admin": true
    }
}
```

<a name="refresh-token"></a>

### トークンをリフレッシュ

トークンは時間的に制限されています。トークンの有効期限が切れた後はアクセスできな
くなります。ただし、期限切れになる前に新しいトークンをリフレッシュすることは可能
です。

ほとんどのアプリケーションはこのエンドポイントを使用して、ユーザがアプリケーショ
ンとインタラクションしている間にユーザのタイムアウトを回避します。

`token` 値は、ユーザが初めてログオンしたときに
、`d848eb12-889f-433b-9811-6a4fbf0b86ca` を取得しました :

#### 3️⃣ リクエスト :

```console
curl -iX POST \
  'http://localhost:3005/v1/auth/tokens' \
  -H 'Content-Type: application/json' \
  -d '{
  "token": "d848eb12-889f-433b-9811-6a4fbf0b86ca"
}'
```

#### レスポンス :

新しいトークンが `X-Subject-Token` ヘッダに返されます :

```
HTTP/1.1 201 Created
X-Subject-Token: a5b83d68-ebad-4514-9d3a-dd892f6e6174
Content-Type: application/json; charset=utf-8
Content-Length: 135
ETag: W/"87-nPb+4XRSsW5Szsf2JJC6UYab4GM"
Date: Mon, 30 Jul 2018 12:41:47 GMT
Connection: keep-alive
```

```json
{
    "token": {
        "methods": ["token"],
        "expires_at": "2018-07-30T13:13:20.567Z"
    },
    "idm_authorization_config": {
        "level": "basic",
        "authzforce": false
    }
}
```

<a name="administrating-user-accounts"></a>

# ユーザ・アカウントの管理

ユーザ・アカウントは、ID 管理システムの中心にあります。すべてのアカウントの必須
フィールドには、ユーザを識別するための一意のユーザ名と電子メールアドレスと、認証
用のパスワードが含まれています。その他のオプションのフィールドには、ユーザの Web
サイト、説明、アバターなど、ユーザに関する詳細情報が追加されます。

デフォルトのスーパー管理ユーザ `admin@test.com` (パスワードは `1234`) として、一
連のユーザ・アカウントを設定し、システム内の関連組織に割り当てます。

<a name="dramatis-personae"></a>

### 登場人物 (Dramatis Personae)

次の人々は、アプリケーション内に正当なアカウントを持っています。

-   Alice, 彼女は **Keyrock** アプリケーションの管理者になります
-   Bob, スーパー・マーケット・チェーンの地域マネージャ。彼の下に数人のマネージ
    ャがいます :
    -   Manager1
    -   Manager2
-   Charlie, スーパー・マーケットチェーン・のセキュリティ責任者。彼の下に数人の
    警備員がいます。
    -   Detective1
    -   Detective2

<a name="user-crud-actions"></a>

## ユーザ CRUD アクション

#### GUI

ユーザは GUI を使用して自分でサインアップすることができます。唯一の要件は電子メ
ールアドレスとパスワードです。

![](https://fiware.github.io/tutorials.Identity-Management/img/sign-up.png)

アカウントが作成されると、そのアカウントの存在を確認しアカウントを有効にするため
に、ユーザに電子メールが送信されます。

![](https://fiware.github.io/tutorials.Identity-Management/img/email.png)

#### REST API

REST API は、独自のやり取りをせずにユーザを作成したり修正したりすることもできま
す。これは、たとえば、大量の CRUD アクションに役立ちます。

> 注 \*\* 招待状を適切に送信するように eMail サーバを設定する必要があります。そ
> うしないと、招待状が迷惑メールとして削除される可能性があります。テストの目的で
> は、users テーブルを直接更新する方が簡単です : `update user set enabled = 1;`

ユーザのためのすべての CRUD アクションでは、以前にログインした管理ユーザからの
`X-Auth-token` ヘッダを使用して、他のユーザ・アカウントを読み取りまたは変更でき
るようにする必要があります。標準の CRUD アクションは、`/v1/users` エンドポイント
の下の適切な HTTP 動詞 (POST, GET, PATCH および DELETE) に割り当てられます。

<a name="creating-users"></a>

### ユーザを作成

新しいユーザを作成するために、以前にログインした管理者の `X-Auth-token` ヘッダと
ともに、ユーザ名、電子メール、パスワードを含む POST リクエストを `/v1/users` エ
ンドポイントに送信します。

#### 4️⃣ リクエスト :

```console
curl -iX POST \
  'http://localhost:3005/v1/users' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
  "user": {
    "username": "alice",
    "email": "alice@test.com",
    "password": "test"
  }
}'
```

#### レスポンス :

レスポンスは、作成されたユーザの詳細を返します :

```
{
    "user": {
        "id": "3b3a5ad5-afd3-4baa-a538-25c7fe7cbf6a",
        "image": "default",
        "gravatar": false,
        "enabled": true,
        "admin": false,
        "starters_tour_ended": false,
        "username": "alice",
        "email": "alice@test.com",
        "date_password": "2018-07-30T12:51:26.813Z"
    }
}
```

新しく作成されたユーザ・アカウントにスーパー管理者権限を与えるには、データベース
を直接変更することができます :

```sql
update user set admin = 1 where username='alice';
```

追加のユーザは、POST リクエストを繰り返すことで追加できます。

たとえば、Bob、地域マネージャ、Charlie、セキュリティ責任者、彼の部下の追加アカウ
ントを作成するには :

```console
curl -iX POST \
  'http://localhost:3005/v1/users' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
  "user": {
    "username": "bob",
    "email": "bob-the-manager@test.com",
    "password": "test"
  }
}'
```

```console
curl -iX POST \
  'http://localhost:3005/v1/users' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
  "user": {
    "username": "charlie",
    "email": "charlie-security@test.com",
    "password": "test"
  }
}'
```

```console
curl -iX POST \
  'http://localhost:3005/v1/users' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
  "user": {
    "username": "manager1",
    "email": "manager1@test.com",
    "password": "test"
  }
}'
```

```console
curl -iX POST \
  'http://localhost:3005/v1/users' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
  "user": {
    "username": "manager2",
    "email": "manager2@test.com",
    "password": "test"
  }
}'
```

```console
curl -iX POST \
  'http://localhost:3005/v1/users' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
  "user": {
    "username": "detective1",
    "email": "detective1@test.com",
    "password": "test"
  }
}'
```

```console
curl -iX POST \
  'http://localhost:3005/v1/users' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
  "user": {
    "username": "detective2",
    "email": "detective2@test.com",
    "password": "test"
  }
}'
```

<a name="read-information-about-a-user"></a>

### ユーザ情報を取得

`/v1/users/{{user-id}}` エンドポイントの下のリソースに GET リクエストを行うと、
その id の下にリストされているユーザが返されます。`X-Auth-token` をヘッダに設定
してください。

#### 5️⃣ リクエスト :

To request

```console
curl -X GET \
  'http://localhost:3005/v1/users/{{user-id}}' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```

#### レスポンス :

レスポンスには、問い合わせたのアカウントの基本的な詳細が含まれています :

```json
{
    "user": {
        "id": "96154659-cb3b-4d2d-afef-18d6aec0518e",
        "username": "alice",
        "email": "alice-the-admin@test.com",
        "enabled": true,
        "admin": false,
        "image": "default",
        "gravatar": false,
        "date_password": "2018-07-30T09:56:37.000Z",
        "description": null,
        "website": null
    }
}
```

<a name="list-all-users"></a>

### すべてのユーザの一覧を取得

すべてのユーザの完全なリストを取得するには、`X-Auth-token` を必要とする、スーパ
ー管理者 (super-admin) の権限が必要です。ほとんどのユーザは、自分の組織内のユー
ザのみを返すことができます。ユーザのリストの取得は、`/v1/users` エンドポイントへ
の GET リクエストを行うことで実行できます。

#### 6️⃣ リクエスト :

```console
curl -X GET \
  'http://localhost:3005/v1/users' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```

#### レスポンス :

```json
{
    "users": [
        {
            "id": "06a2140f-ccc3-49e5-82a5-76bae48b38ba",
            "username": "alice",
            "email": "alice-the-admin@test.com",
            "enabled": true,
            "gravatar": false,
            "date_password": "2018-07-30T11:41:14.000Z",
            "description": null,
            "website": null
        },
        {
            "id": "27e6ae58-adc1-4aaf-a6a2-f207946ba57e",
            "username": "bob",
            "email": "bob-the-manager@test.com",
            "enabled": true,
            "gravatar": false,
            "date_password": "2018-07-30T10:01:12.000Z",
            "description": null,
            "website": null
        },
        ...etc
    ]
}
```

<a name="update-a-user"></a>

### ユーザを更新

すべてのユーザの完全なリストを取得するには、`X-Auth-token` を必要とする、スーパ
ー管理者 (super-admin) の権限が必要です。ほとんどのユーザは、自分の組織内のユー
ザのみを返すことができます。ユーザのリストの取得は、`/v1/users` エンドポイントへ
の GET リクエストを行うことで実行できます。

#### 7️⃣  リクエスト :

```console
curl -iX PATCH \
  'http://localhost:3005/v1/users/{{user-id}}' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
  "user": {
      "username": "alice",
      "email": "alice-the-admin@test.com",
      "enabled": true,
      "gravatar": false,
      "date_password": "2018-07-26T15:25:14.000Z",
      "description": "Alice works for FIWARE",
      "website": "http://www.fiware.org"
  }
}'
```

#### レスポンス :

レスポンスには、更新されたフィールドの一覧が表示されます :

```json
{
    "values_updated": {
        "description": "Alice works for FIWARE",
        "website": "http://www.fiware.org"
    }
}
```

<a name="delete-a-user"></a>

### ユーザを削除

GUI 内で、ユーザは設定ページから自分のアカウントを削除し、**Cancel Account** オ
プションを選択することができます。スーパー管理者のユーザはコマンドラインから
DELETE リクエストを `/v1/users/{{user-id}}` エンドポイントに送信することでこれを
実行できます。`X-Auth-token` ヘッダは、設定されなければなりません。

#### 8️⃣  リクエスト :

```console
curl -iX DELETE \
  'http://localhost:3005/v1/users/{{user-id}}' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```

---

<a name="grouping-user-accounts-under-organizations"></a>

# 組織下でのユーザ・アカウントのグルーピング

合理的なサイズの ID 管理システムでは、ユーザのグループに個々に設定するのではなく
、ロールを割り当てることができると便利です。ユーザ管理は時間のかかるビジネスであ
るため、これらのユーザのグループを管理する責任を、より低いレベルのアクセス権を持
つ他のアカウントに委譲することもできる必要があります。

私たちのスーパー・マーケット・チェーンを考えてみると、店内で商品の価格を変えるこ
とができるユーザ (マネージャ) と、閉店後にドアをロック/ロック解除することができ
るユーザ (店舗の警備員) があります。個々のアカウントにアクセスするのではなく、組
織に権利を割り当てて、ユーザをグループに追加する方が簡単です。

さらに、Alice は、**Keyrock** の管理者が各組織にユーザを明示的に追加する必要はな
く、各組織内の管理者にその権限を委任することができます。たとえば、地域マネージャ
ー の Bob は*管理*組織のオーナーになり、manager1 や manager2 などの追加マネージ
ャーのアカウントをその組織に追加したり削除したりすることができましたが、セキュリ
ティ責任者の Charlie は*セキュリティ*組織の所有ロールを引き受け、その組織に追加
の店舗の警備員を追加することができました。

Bob には*セキュリティ*組織のメンバーシップ・リストを変更する権利がなく、Charlie
には*管理*組織のメンバーシップのリストを変更する権利がないことに注意してください
。 さらに、Bob や Charlie のどちらも、アプリケーション自体の権限を変更することは
できず、既存のユーザ・アカウントを管理している組織に追加して削除するだけで済みま
す。

アプリケーションの作成と権限の設定は、次のチュートリアルの対象であるため、説明し
ません。

<a name="organization-crud-actions"></a>

## 組織 CRUD アクション

#### GUI

サインインすると、ユーザは組織を作成して更新することができます。

![](https://fiware.github.io/tutorials.Identity-Management/img/create-org.png)

#### REST API

または、標準の CRUD アクションは、`/v1/organizations` エンドポイントの下の適切な
HTTP 動詞 (POST, GET, PATCH および DELETE) に割り当てられます。

<a name="create-an-organization"></a>

### 組織の作成

新しい組織を作成するために、以前にログインしたユーザの `X-Auth-token` ヘッダと共
に、`name` と `description` を含む POST リクエストを `/v1/organizations` エンド
ポイントに送信します。

#### 9️⃣ リクエスト :

```console
curl -iX POST \
  'http://localhost:3005/v1/organizations' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
  "organization": {
    "name": "Security",
    "description": "This group is for the store detectives"
  }
}'
```

#### レスポンス :

組織が作成され、それを作成したユーザが自動的にユーザとして割り当てられます。レス
ポンスは新しい組織を識別するために UUID を返します。

```json
{
    "organization": {
        "id": "18deea43-e12a-4018-a45a-664c3158780d",
        "image": "default",
        "name": "Security",
        "description": "This group is for the store detectives"
    }
}
```

<a name="read-organization-details"></a>

### 組織の詳細を取得

`/v1/organizations/{{organization-id}}` エンドポイントの下のリソースに GET リク
エストを行うと、その id の下にリストされている組織が返されます。`X-Auth-token`
は、許可された組織のみが表示されるため、ヘッダに指定する必要があります。

#### 1️⃣0️⃣ リクエスト :

```console
curl -X GET \
  'http://localhost:3005/v1/organizations/{{organization-id}}' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```

#### レスポンス :

レスポンスは組織の詳細を返します。

```json
{
    "organization": {
        "id": "18deea43-e12a-4018-a45a-664c3158780d",
        "name": "Security",
        "description": "This group is for the store detectives",
        "website": null,
        "image": "default"
    }
}
```

<a name="list-all-organizations"></a>

### すべての組織の一覧を取得

すべてのユーザの完全なリストを取得するには、`X-Auth-token` を必要とするスーパー
管理者権限が必要です。ほとんどのユーザは、自分の組織内のユーザのみを返すことが許
可されます。`/v1/organizations` エンドポイントへの GET リクエストを行うことで、
ユーザを取得することができます。

#### 1️⃣1️⃣ リクエスト :

```console
curl -X GET \
  'http://localhost:3005/v1/organizations' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```

#### レスポンス :

レスポンスは、ビジブルな組織の詳細を返します。

```json
{
    "organizations": [
        {
            "role": "owner",
            "Organization": {
                "id": "18deea43-e12a-4018-a45a-664c3158780d",
                "name": "Security",
                "description": "This group is for the store detectives",
                "image": "default",
                "website": null
            }
        },
        {
            "role": "owner",
            "Organization": {
                "id": "a45f9b5a-dd23-4d0f-a0d4-e97e2d7431a3",
                "name": "Management",
                "description": "This group is for the store manangers",
                "image": "default",
                "website": null
            }
        }
    ]
}
```

<a name="update-an-organization"></a>

### 組織を更新

既存の組織の詳細を修正するために、PATCH リクエストを
`/v1/organizations/{{organization-id}}` エンドポイントに送信します。

#### 1️⃣2️⃣ リクエスト :

```console
curl -iX PATCH \
  'http://localhost:3005/v1/organizations/{{organization-id}}' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
    "organization": {
        "name": "FIWARE Security",
        "description": "The FIWARE Foundation is the ...",
        "website": "https://fiware.org"
    }
}'
```

#### レスポンス :

レスポンスには、修正されたフィールドのリストが含まれています。

```json
{
    "values_updated": {
        "name": "FIWARE Security",
        "description": "The FIWARE Foundation is the ..",
        "website": "https://fiware.org"
    }
}
```

<a name="delete-an-organization"></a>

### 組織を削除

#### 1️⃣3️⃣ リクエスト :

```console
curl -iX DELETE \
  'http://localhost:3005/v1/organizations/{{organization-id}}' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```

<a name="users-within-an-organization"></a>

## 組織内のユーザ

組織内のユーザは、`owner` または `member` のいずれかのタイプに割り当てられます。
組織のメンバは、組織自体に割り当てられているすべてのロールと権限を継承します。さ
らに、組織のオーナーは、他のメンバやオーナーを追加したり削除したりすることができ
ます。

<a name="add-a-user-as-a-member-of-an-organization"></a>

### 組織のメンバとしてユーザを追加

GUI を使用して組織にユーザを追加するには、まず既存の組織をクリックし、次に
**Manage** ボタンをクリックします :

![](https://fiware.github.io/tutorials.Identity-Management/img/add-user-to-org.png)

組織のメンバとしてユーザを追加するには、オーナーは URL パスに
`<organization-id>` と `<user-id>` を含む PUT リクエストを作成し、ヘッダに
`X-Auth-Token` を使用して自分自身を識別する必要があります。

#### 1️⃣4️⃣ リクエスト :

```console
curl -iX PUT \
  'http://localhost:3005/v1/organizations/{{organization-id}}/users/{{user-id}}/organization_roles/member' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```

#### レスポンス :

レスポンスは、組織内のユーザの現在のロールを示しています。つまり `member` です。

```json
{
    "user_organization_assignments": {
        "role": "member",
        "organization_id": "18deea43-e12a-4018-a45a-664c3158780d",
        "user_id": "5e482345-2c48-410e-ae03-203d67a43cea"
    }
}
```

<a name="add-a-user-as-an-owner-of-an-organization"></a>

### 組織のオーナーとしてユーザを追加

オーナーは、URL パスに `<organization-id>` および `<user-id>` を含め、PUT リクエ
ストを作成してヘッダ内に `X-Auth-Token` を使用して自分自身を識別することによって
、新しいオーナーを作成することもできます。

#### 1️⃣5️⃣ リクエスト :

```console
curl -iX PUT \
  'http://localhost:3005/v1/organizations/{{organization-id}}/users/{{user-id}}/organization_roles/owner' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```

#### レスポンス :

レスポンスは、組織内のユーザの現在のロールを示しています。つまり `member` です。

```json
{
    "user_organization_assignments": {
        "role": "owner",
        "user_id": "5e482345-2c48-410e-ae03-203d67a43cea",
        "organization_id": "18deea43-e12a-4018-a45a-664c3158780d"
    }
}
```

<a name="list-users-within-an-organization"></a>

### 組織内のユーザの一覧を取得

組織内のユーザをリストすることは、`X-Auth-Toke` を必要とする `owner` またはスー
パー管理者の権限です。ユーザの表示は
、`/v1/organizations/{{organization-id}}/users` エンドポイントに対して GET リク
エストを行うことで実行できます。

#### 1️⃣6️⃣ リクエスト :

```console
curl -X GET \
  'http://localhost:3005/v1/organizations/{{organization-id}}/users' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```

#### レスポンス :

レスポンスにはユーザのリストが含まれます。

```json
{
    "organization_users": [
        {
            "user_id": "admin",
            "organization_id": "18deea43-e12a-4018-a45a-664c3158780d",
            "role": "owner"
        },
        {
            "user_id": "5e482345-2c48-410e-ae03-203d67a43cea",
            "organization_id": "18deea43-e12a-4018-a45a-664c3158780d",
            "role": "member"
        }
    ]
}
```

<a name="read-user-roles-within-an-organization"></a>

### 組織内のユーザ・ロールを取得

組織内のユーザのロールを見つけるには
、`/v1/organizations/{{organization-id}}/users/{{user-id}}/organization_roles`
エンドポイントに GET リクエストを送信します。

#### 1️⃣7️⃣  リクエスト :

```console
curl -X GET \
  'http://localhost:3005/v1/organizations/{{organization-id}}/users/{{user-id}}/organization_roles' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```

#### レスポンス :

レスポンスは、指定された `<user-id>` のロールを返します。

```json
{
    "organization_user": {
        "user_id": "5e482345-2c48-410e-ae03-203d67a43cea",
        "organization_id": "18deea43-e12a-4018-a45a-664c3158780d",
        "role": "member"
    }
}
```

<a name="remove-a-user-from-an-organization"></a>

### 組織からユーザを削除

オーナーとスーパー管理者は、削除リクエストを行うことにより、ユーザを組織から削除
することができます。

#### 1️⃣8️⃣  リクエスト :

```console
curl -X DELETE \
  'http://localhost:3005/v1/organizations/{{organization-id}}/users/{{user-id}}/organization_roles/member' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```

<a name="next-steps"></a>

# 次のステップ

高度な機能を追加することで、アプリケーションに複雑さを加える方法を知りたいですか
？このシリーズ
の[他のチュートリアル](https://www.letsfiware.jp/fiware-tutorials)を読むことで見
つけることができます。

---

## License

[MIT](LICENSE) © 2018-2024 FIWARE Foundation e.V.
