<h1 align="center">
    <img src="https://fiware.github.io/tutorials.Step-by-Step/img/fiware-farm.png" />
    <img src="https://img.shields.io/badge/NGSI-LD-d6604d.svg" width="90"/>
    <br/>
    👨‍🌾 👩‍🌾 🐄 🐐 🐑 🐖 🐓 🌻 🥕 🌽
</h1>

## ID 管理

[![FIWARE Security](https://fiware.github.io/catalogue/badges/chapters/security.svg)](https://github.com/FIWARE/catalogue/blob/master/security/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Identity-Management.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
<br/> [![Documentation](https://img.shields.io/readthedocs/ngsi-ld-tutorials.svg)](https://ngsi-ld-tutorials.rtfd.io)

このチュートリアルは、[Keycloak](https://www.keycloak.org/) の入門です。Keycloak は、
FIWARE サービスに **ID 管理** (Identity Management) を導入するオープンソースの
Identity and Access Management ソリューションです。このチュートリアルでは、
後のチュートリアルでロールと権限を割り当てる準備として、ユーザとグループの
作成方法について説明します。

このチュートリアルでは、**Keycloak** 管理コンソール GUI を使用したインタラクションの例と、
**Keycloak** Admin REST API へのアクセスに使用される [cUrl](https://ec.haxx.se/) コマンドを示します。

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://github.com/codespaces/new?repo=FIWARE/tutorials.Identity-Management&ref=NGSI-LD)

-   This tutorial is also available in [English](README.md).

# コンテンツ

<details>
<summary>詳細 <b>(クリックして拡大)</b></summary>

-   [ID 管理](#id-管理)
    -   [ID 管理の標準概念](#id-管理の標準概念)
-   [前提条件](#前提条件)
    -   [Docker](#docker)
    -   [WSL](#wsl)
-   [アーキテクチャ](#アーキテクチャ)
    -   [Keycloak の設定](#keycloak-の設定)
    -   [PostgreSQL の設定](#postgresql-の設定)
-   [起動](#起動)
    -   [Keycloak Admin API の参照](#keycloak-admin-api-の参照)
    -   [Keycloak 内の UUID](#keycloak-内の-uuid)
    -   [ログイン](#ログイン)
        -   [管理者トークンを取得](#管理者トークンを取得)
        -   [トークンからユーザ情報を取得](#トークンからユーザ情報を取得)
        -   [トークンをリフレッシュ](#トークンをリフレッシュ)
-   [ユーザ・アカウントの管理](#ユーザアカウントの管理)
    -   [登場人物 (Dramatis Personae)](#登場人物-dramatis-personae)
    -   [ユーザ CRUD アクション](#ユーザ-crud-アクション)
        -   [ユーザを作成](#ユーザを作成)
        -   [ユーザ情報を取得](#ユーザ情報を取得)
        -   [すべてのユーザの一覧を取得](#すべてのユーザの一覧を取得)
        -   [ユーザを更新](#ユーザを更新)
        -   [ユーザを削除](#ユーザを削除)
-   [グループ下でのユーザ・アカウントのグルーピング](#グループ下でのユーザアカウントのグルーピング)
    -   [グループ CRUD アクション](#グループ-crud-アクション)
        -   [グループを作成](#グループを作成)
        -   [グループの詳細を取得](#グループの詳細を取得)
        -   [すべてのグループの一覧を取得](#すべてのグループの一覧を取得)
        -   [グループを更新](#グループを更新)
        -   [グループを削除](#グループを削除)
    -   [グループ内のユーザ](#グループ内のユーザ)
        -   [ユーザをグループのメンバーとして追加](#ユーザをグループのメンバーとして追加)
        -   [グループ内のユーザを一覧表示](#グループ内のユーザを一覧表示)
        -   [グループからユーザを削除](#グループからユーザを削除)
-   [次のステップ](#次のステップ)

</details>

# ID 管理

> "権力を持つ人物に出会ったならば、5 つの質問をすべきだ: 'あなたはどんな権力を持っているのか？
> どこからそれを得たのか？誰の利益のためにそれを行使するのか？誰に対して責任を負うのか？
> そしてどうすればあなたを排除できるのか？'"
>
> — アンソニー・ウェッジウッド・ベン (民主主義の 5 つの本質的な質問)

コンピュータセキュリティの用語では、ID 管理とは「適切な人物が適切な時に適切な理由で適切なリソースに
アクセスできるようにする」セキュリティおよびビジネスの規律です。<sup>[1](#footnote1)</sup>
これは、異なるシステム間でリソースへの適切なアクセスを確保する必要性に対応するものです。

FIWARE フレームワークは一連の独立したコンポーネントで構成されており、セキュリティ章では
これらのコンポーネントに共通するアクセス制御の要件を実装することを目指しています。
リソースへのアクセスを制限する前に、リクエストを行う人物 (またはサービス) の ID を
確認する必要があります。[Keycloak](https://www.keycloak.org/) はオープンソースの
Identity and Access Management ソリューションであり、OpenID Connect と OAuth 2.0 の
業界標準プロトコルに基づいて、すべての共通 ID 管理機能をすぐに利用できる形で提供します。

## ID 管理の標準概念

**Keycloak** Identity Management システムには、以下の共通オブジェクトが存在します:

-   **ユーザ (User)** — ユーザ名とパスワードで識別できる登録済みユーザ。ユーザには個別に、
    またはグループの一部として権限を付与できます。
-   **レルム (Realm)** — ユーザ、認証情報、ロール、グループのセットを管理するセキュリティドメイン。
    レルムは他のレルムから分離されており、管理するリソースのみを制御します。このチュートリアルでは
    レルム名を `farm-management` とします。
-   **グループ (Group)** — ロールのセットを割り当てることができるユーザのコレクション。
    グループのロール割り当てを変更すると、そのグループのすべてのメンバーのアクセス権が影響を受けます。
-   **ロール (Role)** — 権限セットのラベル。ロールは個別のユーザまたはグループに割り当てることができます。
    サインインしたユーザは、自身のロールとグループから継承したロールのすべての権限を取得します。
-   **クライアント (Client)** — 認証をリクエストできるアプリケーションまたはサービス。
    後のチュートリアルでは、NGSI-LD コンテキストブローカープロキシが `farm-management` レルムの
    クライアントとして登録されます。

# 前提条件

## Docker

シンプルさを維持するため、すべてのコンポーネントは [Docker](https://www.docker.com) を使用して実行されます。
**Docker** は、異なるコンポーネントをそれぞれの環境に分離できるコンテナ技術です。

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義および実行するためのツールです。

現在の **Docker** と **Docker Compose** のバージョンは、以下のコマンドで確認できます:

```console
docker version
docker compose version
```

Docker バージョン 24.0 以上、Docker Compose バージョン 2.24 以上を使用していることを確認し、
必要に応じてアップグレードしてください。

## WSL

Windows Subsystem for Linux を使用すると、Windows 上でネイティブに Linux バイナリを実行できます。
Windows をお使いで Linux 経由でチュートリアルを実行したい場合は、WSL2 をインストールして
すべての保留中の更新を適用してから開始してください。

# アーキテクチャ

このチュートリアルでは、[Keycloak](https://www.keycloak.org/) と **PostgreSQL** データベースを使用します。

アーキテクチャ全体は以下の要素で構成されます:

-   **Identity and Access Management** コンポーネント:
    -   [Keycloak](https://www.keycloak.org/) は以下を含む完全な ID 管理システムを提供します:
        -   アプリケーションとユーザの認証・認可サーバ
        -   ID 管理管理のための管理コンソール GUI
        -   HTTP による ID 管理のための完全な REST API
-   [PostgreSQL](https://www.postgresql.org/) データベース:
    -   Keycloak レルムデータ、ユーザ、グループ、ロール、セッションの永続化に使用

## Keycloak の設定

```yaml
keycloak:
    image: quay.io/keycloak/keycloak:24.0.1
    container_name: fiware-keycloak
    hostname: keycloak
    ports:
        - "3005:8080"
    environment:
        - KC_DB=postgres
        - KC_DB_URL=jdbc:postgresql://postgres-keycloak/keycloak
        - KC_DB_USERNAME=keycloak
        - KC_DB_PASSWORD=password
        - KC_BOOTSTRAP_ADMIN_USERNAME=admin
        - KC_BOOTSTRAP_ADMIN_PASSWORD=1234
        - KC_HTTP_PORT=8080
        - KC_HOSTNAME_STRICT=false
        - KC_HTTP_ENABLED=true
        - KC_HEALTH_ENABLED=true
    command: start-dev --import-realm
    volumes:
        - ./realm-config:/opt/keycloak/data/import:ro
```

`keycloak` コンテナは内部でポート `8080` をリッスンする Web アプリケーションサーバです。
ホストにはポート `3005` が公開されており、ブラウザと REST API アクセスが可能です。

主要な環境変数:

| キー                           | 値                                                  | 説明                                                              |
| ------------------------------ | --------------------------------------------------- | ----------------------------------------------------------------- |
| `KC_DB`                        | `postgres`                                          | データベースタイプ — この設定では PostgreSQL が必要               |
| `KC_BOOTSTRAP_ADMIN_USERNAME`  | `admin`                                             | マスターレルムの初期管理者ユーザ名                                |
| `KC_BOOTSTRAP_ADMIN_PASSWORD`  | `1234`                                              | 初期管理者パスワード — 本番環境では直ちに変更すること             |
| `KC_HEALTH_ENABLED`            | `true`                                              | Docker ヘルスチェックで使用される `/health/ready` エンドポイントを有効にする |

`start-dev` コマンドは Keycloak を開発モードで起動します。`--import-realm` フラグにより、
起動時に `/opt/keycloak/data/import/` ディレクトリ内のレルム JSON ファイルがインポートされます。

## PostgreSQL の設定

```yaml
postgres-keycloak:
    image: postgres:15
    container_name: db-postgres
    hostname: postgres-keycloak
    environment:
        - POSTGRES_DB=keycloak
        - POSTGRES_USER=keycloak
        - POSTGRES_PASSWORD=password
```

`postgres-keycloak` コンテナはデフォルトポート `5432` でリッスンします (内部のみ — ホストには公開されていません)。

# 起動

インストールを開始するには、以下を実行します:

```console
git clone https://github.com/FIWARE/tutorials.Identity-Management.git
cd tutorials.Identity-Management
git checkout NGSI-LD

./services create
./services start
```

> [!NOTE]
>
> クリーンアップして再起動する場合は、以下のコマンドを実行してください:
>
> ```console
> ./services stop
> ```

## Keycloak Admin API の参照

すべての ID 管理レコードは Keycloak Admin REST API を使用して照会できます。実行後、API は
`http://localhost:3005/admin/realms/` でアクセスできます。管理コンソール GUI は
`http://localhost:3005` で利用できます。

すべての Admin API 呼び出しには、`Authorization` ヘッダに管理者 `Bearer` トークンが必要です。

## Keycloak 内の UUID

**Keycloak** 内のすべての ID は作成時に生成される UUID であり、その後変更されません。
このチュートリアル全体で使用されるプレースホルダ値を、実際の ID に置き換える必要があります:

| キー           | 説明                                              | サンプル値                                      |
| -------------- | ------------------------------------------------- | ----------------------------------------------- |
| `{{token}}`    | マスターレルムから取得した管理者 Bearer トークン  | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVC...`         |
| `{{user-id}}`  | `farm-management` レルム内の既存ユーザの UUID     | `96154659-cb3b-4d2d-afef-18d6aec0518e`          |
| `{{group-id}}` | `farm-management` レルム内の既存グループの UUID   | `74f5299e-3247-468c-affb-957cda03f0c4`          |

## ログイン

`http://localhost:3005` の管理コンソールが Keycloak 管理の主要 GUI です。
ユーザ名 `admin`、パスワード `1234` でログインして、マスターレルムにアクセスします。
`farm-management` レルムで作業するには、左上のレルムドロップダウンから選択してください。

### 管理者トークンを取得

以下のリクエストは管理者認証情報を使用してマスターレルムにログインし、JWT アクセストークンを返します。

#### 1️⃣ Request:

```console
curl -iX POST \
  'http://localhost:3005/realms/master/protocol/openid-connect/token' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --data-urlencode 'grant_type=password' \
  --data-urlencode 'client_id=admin-cli' \
  --data-urlencode 'username=admin' \
  --data-urlencode 'password=1234'
```

#### Response:

レスポンスには `access_token` (署名済み JWT)、`refresh_token`、有効期限情報が含まれます。

```json
{
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJhZ...",
    "expires_in": 60,
    "refresh_expires_in": 1800,
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJh...",
    "token_type": "Bearer"
}
```

### トークンからユーザ情報を取得

Keycloak の UserInfo エンドポイントを使用すると、ローカルの JWT デコードなしにトークンのクレームを取得できます。

#### 2️⃣ Request:

```console
curl -X GET \
  'http://localhost:3005/realms/farm-management/protocol/openid-connect/userinfo' \
  -H 'Authorization: Bearer {{token}}'
```

#### Response:

```json
{
    "sub": "3b3a5ad5-afd3-4baa-a538-25c7fe7cbf6a",
    "preferred_username": "alice",
    "given_name": "Alice",
    "family_name": "Administrator",
    "email": "alice@fiware.farm"
}
```

### トークンをリフレッシュ

トークンには有効期限があります。Request 1️⃣ で返された `refresh_token` を使用して、
ユーザの再認証なしに新しいアクセストークンと交換できます。

#### 3️⃣ Request:

```console
curl -iX POST \
  'http://localhost:3005/realms/master/protocol/openid-connect/token' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --data-urlencode 'grant_type=refresh_token' \
  --data-urlencode 'client_id=admin-cli' \
  --data-urlencode 'refresh_token={{refresh_token}}'
```

#### Response:

更新された有効期限を持つ新しい `access_token` と `refresh_token` のペアが返されます。

# ユーザ・アカウントの管理

ユーザアカウントはあらゆる ID 管理システムの中心です。各アカウントには、ユーザを識別するための
一意のユーザ名とメールアドレス、および認証用のパスワードが含まれます。

デフォルトのスーパー管理者ユーザ `admin` (パスワード `1234`) として、一連のユーザアカウントを
設定し、システム内の関連グループに割り当てます。

### 登場人物 (Dramatis Personae)

以下の人物が農場管理アプリケーション内で正当にアカウントを持っています:

-   **Alice** — システム管理者。すべての Keycloak 設定を管理します
-   **Bob** — 農場オーナー兼ゼネラルマネージャー。すべての農場コンテキストデータへのフルアクセス
-   **Carol** — 畜産マネージャー。以下のメンバーを監督します:
    -   **Frank** — 畜産フィールドワーカー
    -   **Grace** — 畜産フィールドワーカー
-   **Dave** — 作物・灌漑マネージャー。以下のメンバーを監督します:
    -   **Harry** — 灌漑オペレーター
-   **Eve** — 設備マネージャー。以下のメンバーを監督します:
    -   **Ivy** — トラクターオペレーター
-   **Jenny** — 外部獣医師。動物の健康データへの読み取り専用アクセス
-   **Ken** — 外部農学者。土壌・作物データへの読み取り専用アクセス

## ユーザ CRUD アクション

#### GUI

ユーザは `http://localhost:3005` の管理コンソールで作成できます。

**レルム: farm-management → ユーザ → ユーザを追加** に移動します。

#### REST API

すべてのユーザ CRUD アクションには、事前に取得した管理者トークンの
`Authorization: Bearer {{token}}` ヘッダが必要です。

### ユーザを作成

#### 4️⃣ Request:

```console
curl -iX POST \
  'http://localhost:3005/admin/realms/farm-management/users' \
  -H 'Authorization: Bearer {{token}}' \
  -H 'Content-Type: application/json' \
  -d '{
    "username": "alice",
    "firstName": "Alice",
    "lastName": "Administrator",
    "email": "alice@fiware.farm",
    "enabled": true,
    "credentials": [{"type": "password", "value": "test", "temporary": false}]
  }'
```

#### Response:

成功時には `Location` ヘッダに新規作成されたユーザの URL を含む `201 Created` レスポンスが返されます。

```
HTTP/1.1 201 Created
Location: http://localhost:3005/admin/realms/farm-management/users/3b3a5ad5-afd3-4baa-a538-25c7fe7cbf6a
```

### ユーザ情報を取得

#### 5️⃣ Request:

```console
curl -X GET \
  'http://localhost:3005/admin/realms/farm-management/users/{{user-id}}' \
  -H 'Authorization: Bearer {{token}}'
```

#### Response:

```json
{
    "id": "3b3a5ad5-afd3-4baa-a538-25c7fe7cbf6a",
    "username": "alice",
    "enabled": true,
    "firstName": "Alice",
    "lastName": "Administrator",
    "email": "alice@fiware.farm"
}
```

### すべてのユーザの一覧を取得

#### 6️⃣ Request:

```console
curl -X GET \
  'http://localhost:3005/admin/realms/farm-management/users' \
  -H 'Authorization: Bearer {{token}}'
```

### ユーザを更新

#### 7️⃣ Request:

```console
curl -iX PUT \
  'http://localhost:3005/admin/realms/farm-management/users/{{user-id}}' \
  -H 'Authorization: Bearer {{token}}' \
  -H 'Content-Type: application/json' \
  -d '{
    "username": "alice",
    "firstName": "Alice",
    "lastName": "Administrator",
    "email": "alice.admin@fiware.farm",
    "enabled": true
  }'
```

#### Response:

`204 No Content` レスポンスは更新が正常に適用されたことを示します。

### ユーザを削除

#### 8️⃣ Request:

```console
curl -iX DELETE \
  'http://localhost:3005/admin/realms/farm-management/users/{{user-id}}' \
  -H 'Authorization: Bearer {{token}}'
```

# グループ下でのユーザ・アカウントのグルーピング

Keycloak では、**グループ** はロールをまとめて割り当てることができるユーザのコレクションです。
グループにロールが割り当てられると、そのグループのすべてのメンバーがそのロールを継承します。

`farm-management` レルムはレルムインポートファイルから以下のグループで事前設定されています:

| グループ名               | 説明                                   |
| ------------------------ | -------------------------------------- |
| `farm-management`        | 農場オーナーと一般管理スタッフ         |
| `livestock-team`         | 畜産監督者とフィールドワーカー         |
| `crop-team`              | 作物・灌漑監督者とワーカー             |
| `equipment-team`         | 設備・機械監督者                       |
| `external-consultants`   | 読み取り専用アクセスを持つ外部専門家   |

## グループ CRUD アクション

### グループを作成

#### 9️⃣ Request:

```console
curl -iX POST \
  'http://localhost:3005/admin/realms/farm-management/groups' \
  -H 'Authorization: Bearer {{token}}' \
  -H 'Content-Type: application/json' \
  -d '{"name": "drone-operators"}'
```

#### Response:

```
HTTP/1.1 201 Created
Location: http://localhost:3005/admin/realms/farm-management/groups/e424ed98-c966-46e3-b161-a165fd31bc01
```

### グループの詳細を取得

#### 1️⃣0️⃣ Request:

```console
curl -X GET \
  'http://localhost:3005/admin/realms/farm-management/groups/{{group-id}}' \
  -H 'Authorization: Bearer {{token}}'
```

### すべてのグループの一覧を取得

#### 1️⃣1️⃣ Request:

```console
curl -X GET \
  'http://localhost:3005/admin/realms/farm-management/groups' \
  -H 'Authorization: Bearer {{token}}'
```

### グループを更新

#### 1️⃣2️⃣ Request:

```console
curl -iX PUT \
  'http://localhost:3005/admin/realms/farm-management/groups/{{group-id}}' \
  -H 'Authorization: Bearer {{token}}' \
  -H 'Content-Type: application/json' \
  -d '{"name": "livestock-and-dairy-team"}'
```

### グループを削除

#### 1️⃣3️⃣ Request:

```console
curl -iX DELETE \
  'http://localhost:3005/admin/realms/farm-management/groups/{{group-id}}' \
  -H 'Authorization: Bearer {{token}}'
```

グループが削除されても、グループのメンバーは削除されません。
メンバーはグループレベルで割り当てられていたロールを失うだけです。

## グループ内のユーザ

### ユーザをグループのメンバーとして追加

#### 1️⃣4️⃣ Request:

```console
curl -iX PUT \
  'http://localhost:3005/admin/realms/farm-management/users/{{user-id}}/groups/{{group-id}}' \
  -H 'Authorization: Bearer {{token}}'
```

#### Response:

`204 No Content` レスポンスはユーザがグループに追加されたことを示します。

### グループ内のユーザを一覧表示

#### 1️⃣5️⃣ Request:

```console
curl -X GET \
  'http://localhost:3005/admin/realms/farm-management/groups/{{group-id}}/members' \
  -H 'Authorization: Bearer {{token}}'
```

### グループからユーザを削除

#### 1️⃣6️⃣ Request:

```console
curl -iX DELETE \
  'http://localhost:3005/admin/realms/farm-management/users/{{user-id}}/groups/{{group-id}}' \
  -H 'Authorization: Bearer {{token}}'
```

# 次のステップ

高度な機能を追加してアプリケーションをさらに複雑にする方法を学びたい場合は、
他の [NGSI-LD チュートリアル](https://ngsi-ld-tutorials.rtfd.io) をお読みください。

---

<a name="footnote1"></a>
1. [OASIS](https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=idtrust) 標準からの定義を参照。
