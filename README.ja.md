# IoT Sensors[<img src="https://img.shields.io/badge/NGSI-LD-d6604d.svg" width="90"  align="left" />](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.08.01_60/gs_cim009v010801p.pdf)[<img src="https://fiware.github.io/tutorials.IoT-Sensors/img/fiware.png" align="left" width="162">](https://www.fiware.org/)<br/>

[![FIWARE IoT Agents](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/iot-agents.svg)](https://github.com/FIWARE/catalogue/blob/master/iot-agents/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.IoT-Sensors.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
[![UltraLight 2.0](https://img.shields.io/badge/Payload-Ultralight-27ae60.svg)](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

このチュートリアルは、IoT デバイスの概要と、制約のあるデバイスでの
[UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
プロトコルの使用法です。このチュートリアルでは、一連のダミー農業 IoT デバイスを紹介します。これは、ブラウザー内に表示され、
ユーザがそれらを操作できるようにします。実際の IoT Agent を介して IoT デバイスを NGSI-LD Context Broker に接続する前に、
このチュートリアルで定義されているすべての用語と概念を完全に理解する必要があります。

このチュートリアルでは、全体で [cUrl](https://ec.haxx.se/) コマンドを使用していますが、
[Postman documentation](https://fiware.github.io/tutorials.IoT-Sensors/ngsi-ld.html) のドキュメントとしても利用できます。

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/1805fa92c4d6abaa374f)
[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/FIWARE/tutorials.IoT-Sensors/tree/NGSI-LD)

## コンテンツ

<details>
<summary><strong>詳細</strong></summary>

-   [IoT デバイスとは何ですか ?](#what-are-iot-devices)
-   [Ultralight 2.0 とは何ですか ?](#what-is-ultralight-20)
    -   [サウスバウンド・トラフィック (コマンド)](#southbound-traffic-commands)
        -   [HTTP POST を使用したコマンドのプッシュ](#push-command-using-http-post)
    -   [ノースバウンド・トラフィック (測定値)](#northbound-traffic-measurements)
        -   [HTTP GET を使用した測定](#measurement-using-http-get)
        -   [HTTP POST を使用した測定](#measurement-using-http-post)
-   [アーキテクチャ](#architecture)
-   [前提条件](#prerequisites)
    -   [Docker](#docker)
    -   [WSL](#wsl)
-   [起動](#start-up)
-   [IoT デバイスとの通信](#communicating-with-iot-devices)
    -   [Irrigation System (灌漑システム) のコマンド](#irrigation-system-commands)
        -   [Irrigation System をオンにする](#turn-on-the-irrigation-system)
        -   [Irrigation System をオフにする](#turn-off-the-irrigation-system)
    -   [Tractor ( トラクター) のコマンド](#tractor-commands)
    -   [FMIS System (FMIS システム)](#fmis-system-commands)
        -   [Tractor をアクティブ化](#activate-a-tractor)
        -   [Tractor を非アクティブ化](#deactivate-a-tractor)
    -   [Filling Station (充填ステーション) のコマンド](#filling-station-commands)
        -   [納屋から干し草を取り除く](#remove-hay-from-the-barn)
    -   [測定値の送信](#sending-measures)

</details>

<a name="what-are-iot-devices"/>

# IoT デバイスとは何ですか ?

> "A farm is a manipulative creature. There is no such thing as finished. Work comes in a stream and has no end. There
> are only the things that must be done now and things that can be done later.."
>
> — Kristin Kimball, The Dirty Life: On Farming, Food, and Love

[Internet of Things](https://www.linux.com/news/who-needs-internet-things) (IoT) は、ネットワークに接続してデータを
交換できる物理デバイスのネットワークです。各 "thing" (モノ) または "smart device" (スマート・デバイス) は、センサ
またはアクチュエータとして機能できる電子機器とソフトウェアが組み込まれたガジェットです。センサは、周囲の現実世界の
状態を報告できます。アクチュエータは、制御信号に応答して、システムの状態を変更する責任があります。

各デバイスは、組み込みコンピューティング・システムを通じて一意に識別できますが、
既存のインターネット・インフラストラクチャ内で相互運用できます。

FIWARE は、コンテキスト情報を管理するためのシステムです。Internet of Things に基づくスマート・ソリューションの場合、
コンテキストは一連の接続された IoT デバイスによって提供されます。各 IoT デバイスは実世界に存在する物理オブジェクトで
あるため、最終的にはコンテキスト内で一意のエンティティとして表されます。

IoT デバイスは、単純なものから複雑なものまでさまざまです。
このチュートリアルで使用される農業用 IoT デバイスの例をいくつか示します:

-   **Soil Sensor** (土壌センサ) は、地盤中の水分量を報告することができます
-   **Temperature Sensor** (温度センサ) は、現在の空気や土壌の温度を返すように照会することができます
-   **Filling Sensor** (充填センサ) は、サイロに残された飼料の量を報告することができます
-   **Irrigation System** (灌漑システム) は、コマンドを送信して、短期間アクティブ化してオンにすることができます
-   **Animal Collars** (動物の首輪) は、家畜の場所、健康状態、ストレス・レベルを追跡するために使用できます
-   **Farm Management Information Systems** (農場管理情報システム) は、農業機械の中に設置され、労働者への指示の送受信、
    タスクの状態の追跡、およびそれらの進捗状況の追跡に使用できます

ご覧のとおり、**Irrigation System** (灌漑システム) は特定のコマンドにのみ反応するため、純粋なアクチュエータの
一例です。一方、**Soil Sensor** (土壌センサ) は純粋なセンサの一例です。これは、目にする世界の状態についてのみ
報告するためです。**FMIS** などの一部のデバイスは、コマンドに応答し、意味のある方法で状態を報告することができます。

各デバイス内に保持される状態情報は、最終的に Context Broker 内に保持されるため、次の図で定義されています:

![](https://fiware.github.io/tutorials.IoT-Sensors/img/entities-ld.png)

<a name="what-is-ultralight-20"/>

# Ultralight 2.0 とは何ですか ?

[UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
は、帯域幅とデバイスのメモリ・リソースが制限されている制約されたデバイスと通信用の軽量テキスト・ベースのプロトコルです。
測定リクエストのペイロードは、パイプ `|` 文字で区切られたキーと値のペアのリストです。

例えば、

```
<key>|<value>|<key>|<value>|<key>|<value> etc..
```

たとえば、次のようなペイロード:

```
t|15|k|abc
```

2つの属性が含まれ、1つは値が "15" の "t" という名前で、もう1つは値 "abc" の "k" という名前で送信されます。
Ultralight 2.0 の値はタイプされません (すべてが文字列として扱われます)。

Ultralight 2.0 は、デバイスとサーバ間で共有する測定とコマンドを記述するペイロードを定義しますが、単一のトランスポート・
プロトコルを指定しません。代わりに、さまざまなシナリオでさまざまなトランスポート・プロトコル・バインディング
(HTTP, MQTT, AMQP などを使用できます。このチュートリアルでは、HTTP をトランスポート・プロトコルとして使用します。

<a name="southbound-traffic-commands"/>

## サウスバウンド・トラフィック (コマンド)

Context Broker からによって生成され、IoT デバイスに向けて (IoT Agent 経由で) 下向きに渡される HTTP リクエストは、
サウスバウンド・トラフィックと呼ばれます。サウスバウンド・トラフィックは、アクチュエータ・デバイスに対して行われた
コマンドで構成されており、それらのアクションによって現実世界の状態を変更します。たとえば、ランプの状態を変更するコマンド
`ON` は、実際にランプをオンにします。これにより、近くにある他のセンサの読み取り値が変わる可能性があります。

<a name="push-command-using-http-post"/>

### HTTP POST を使用したコマンドのプッシュ

IoT Agent と IoT デバイス間のサウスバウンド通信のセットアップは、プロビジョニングと呼ばれます。これにより、IoT Agent
が各 IoT デバイスに接続できるように十分な情報を保持することが保証されます。つまり、コマンドの送信先とサポートされている
コマンドを認識しています。コマンドをデバイスに送信するために、IoT Agent は POST リクエストをデバイスが提供する
エンドポイントに送信します。POST リクエストのボディにはコマンドが含まれています。

Ultralight コマンドのペイロードの形式は次のとおりです :

```
<device name>@<command>|<param|<param>
```

ここで `<device_name>` は Context Broker に保持されているエンティティ `id` であり、`<command>` はサポートされている
コマンドの1つであり、追加の必要な値は後続のパラメータで渡されます。たとえば、

```
urn:ngsi-ld:Robot:001@turn|left|30
```

これは、「私は Context Broker 内で `id="urn:ngsi-ld:Robot:001"` として知られています。このエンドポイントをリッスン
しているデバイスに `turn` コマンドを実行してもらいたい。デバイスが操作を実行できるようにするために、必要に応じて
パラメータ `left` および `30` (degrees) を指定しました」とデバイスに送信します

IoT Agent に対する定義済みのノースバウンドのレスポンスは次のとおりです:

```
urn:ngsi-ld:Robot:001@turn|Turn ok
```

これは、「私は、Context Broker 内の `id="urn:ngsi-ld:Robot:001"` として知られているエンティティからのリクエストに
応じました。 私が実行したコマンドは `turn` コマンドでした。結果は `Turn ok` でした」と言っています。

ご覧のとおり、サウスバウンド・コマンドはインタラクション内で使用される `id` を定義し、同じデータも返されるため、
すべてのレスポンスは常に Context Broker 内に保持されている適切なエンティティに関連付けることができます。

プッシュ・コマンドは、デバイスがサウスバウンド・トラフィックをリッスンするための個別のエンドポイントを提供できる
場合にのみ使用できます。すべての対話がデバイス自体から開始される場合、代替のポーリング・メカニズムを使用できますが、
これはこのチュートリアルの範囲外です。

<a name="northbound-traffic-measurements"/>

## ノースバウンド・トラフィック (測定値)

IoT デバイスから生成され、(IoT Agent を介して) Context Broker に向かって上方に渡されるリクエストは、ノースバウンド・
トラフィックと呼ばれます。ノースバウンド・トラフィックは、センサ・デバイスによる測定で構成され、現実世界の状態を
システムのコンテキスト・データに中継します。たとえば、湿度センサの測定値を Context Broker にリレーして、
エンティティの湿度レベルが変化したことを示すことができます。そのような変更について通知されるようにサブスクリプションを
作成すると、追加のアクション (スプリンクラーをオンにするなど) が引き起こされます。

<a name="measurement-using-http-get"/>

### HTTP GET を使用した測定

デバイスは、"既知の" エンドポイント (パス `/iot/d`) への HTTP GET リクエストと次のクエリ・パラメータを使用して、
IoT Agent に新しい測定値を報告できます:

-   `i` (device ID): デバイス ID (API キーに固有)
-   `k` (API Key): デバイスが登録されているサービスの API キー
-   `t` (timestamp): 測定のタイムスタンプ。自動 IoTAgent タイムスタンプを上書きします (オプション)
-   `d` (Data): Ultralight 2.0 ペイロード

`i` と `k` パラメータは必須です。

たとえば、リクエスト:

```
<iot-agent>/iot/d?i=humidity001&d=h|12
```

これは、デバイス `id=motion001` が、値 `12` で実際の測定 `h` を実行したことを IoT Agent に通知したいことを示します。
これは最終的に Context Broker に渡されます。

<a name="measurement-using-http-post"/>

### HTTP POST を使用した測定

HTTP POST も使用できます。この場合もパスは `/iot/d` になりますが、この場合は `d` (Data) は必要ありません。
測定のキーと値のペアがリクエストのボディとして渡されます。`i` と `k` のクエリ・パラメータは必須です:

-   `i` (device ID): デバイス ID (API キーに固有)
-   `k` (API Key): デバイスが登録されているサービスの API キー
-   `t` (timestamp): 測定のタイムスタンプ。自動 IoTAgent タイムスタンプを上書きします (オプション)

ここでも、`i` と `k` パラメータは必須です。

#### デバイス・モニタ

このチュートリアルでは、一連のダミー IoT デバイスが作成され、最終的には Context Broker に接続されます。
各デバイスの状態は、次の場所にある UltraLight デバイス・モニタの Web ページで確認できます:
`http://localhost:3000/device/monitor`

![FIWARE Monitor](https://fiware.github.io/tutorials.IoT-Sensors/img/farm-devices.png)

<a name="architecture"/>

# アーキテクチャ

デモ・アプリケーションは、ダミーの IoT デバイスのセットとして機能する単一のカスタム・コンポーネントのみを使用します。
すべての IoT デバイスは、HTTP で実行される
[UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
プロトコルを使用します。すべての対話は HTTP リクエストによって開始されるため、エンティティをコンテナ化し、
公開されたポートから実行できます。

![](https://fiware.github.io/tutorials.IoT-Sensors/img/architecture-ld.png)

必要な構成情報は、関連する `docker-compose.yml` ファイルの services セクションで確認できます:

```yaml
tutorial:
    image: quay.io/fiware/tutorials.ngsi-ld
    hostname: iot-sensors
    container_name: fiware-tutorial
    networks:
        - default
    expose:
        - "3000"
        - "3001"
    ports:
        - "3000:3000"
        - "3001:3001"
    environment:
        - "DEBUG=tutorial:*"
        - "PORT=3000"
        - "IOTA_HTTP_HOST=iot-agent"
        - "IOTA_HTTP_PORT=7896"
        - "DUMMY_DEVICES_PORT=3001" # Port used by the dummy IOT devices to receive commands
        - "DUMMY_DEVICES_API_KEY=4jggokgpepnvsb2uv4s40d59ov"
```

`tutorial` コンテナは、2つのポートでリッスンしています:

-   Port `3000` は、公開されているので、ダミーの IoT デバイスを表示する Web ページを見ることができます
-   Port `3001` は、純粋にチュートリアル・アクセスのために公開されています。cUrl または Postman
    が同じネットワークに属さなくても UltraLight コマンドを作成できるようにするためです

次に示すように、`tutorial` コンテナは環境変数によって駆動されます:

| キー                  | 値                           | 説明                                                                                                                                                                               |
| --------------------- | ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| DEBUG                 | `tutorial:*`                 | ロギングに使用されるデバッグ・フラグ                                                                                                                                                |
| WEB_APP_PORT          | `3000`                       | ダミー・デバイスのデータを表示する Web アプリが使用するポート                                                                                                                       |
| IOTA_HTTP_HOST        | `iot-agent`                  | 欠落している IoT Agent のホスト名 - 後のチュートリアルで使用                                                                                                                        |
| IOTA_HTTP_PORT        | `7896`                       | 欠落している IoT Agent がリッスンするポート。`7896` は、UltraLight over HTTP の一般的なデフォルトです                                                                               |
| DUMMY_DEVICES_PORT    | `3001`                       | コマンドを受信するためにダミーの IoT デバイスが使用するポート                                                                                                                       |
| DUMMY_DEVICES_API_KEY | `4jggokgpepnvsb2uv4s40d59ov` | UltraLight インタラクションに使用されるランダムなセキュリティキー - これは、後のチュートリアルで使用され、デバイスと欠落している IoT Agent 間のインタラクションの整合性を保証します |

YAML ファイルに記述されている他の `tutorial` コンテナの構成値は、このチュートリアルでは使用しません。

実用的なスマート・ソリューションを介して渡されるメッセージについて説明するとき、このチュートリアルでは使用されない
2つのコンポーネントを参照しますが、後でシステムを完成させるために必要になります。

-   [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) は、スマート・ソリューションのコンテキスト
    ・データを保持するために使用されます。ご存知のように、Context Broker とのすべての対話は
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
    を使用して行う必要があります
-   IoT Agent は、(Context Broker からの)
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
    リクエストを、IoT デバイス自体が使用できるプロトコル (
    [UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
    など) に変換するミドルウェア・コンポーネントとして機能します

したがって、最初にサンプルのデバイス・プロトコルを理解し、メッセージがシステムをどのように通過して、IoT Agent
ミドルウェアの目的を理解するかを完全に理解する必要があります。このチュートリアルでは、デバイスにコマンドを発行し、
デバイスから測定値を受け取る IoT Agent の役割を果たします。

<a name="prerequisites"/>

# 前提条件

<a name="docker"/>

## Docker

物事を単純にするために、両方のコンポーネントが [Docker](https://www.docker.com) を使用して実行されます。**Docker**
は、さまざまコンポーネントをそれぞれの環境に分離することを可能にするコンテナ・テクノロジです。

-   Docker を Windows にインストールするには
    、[こちら](https://docs.docker.com/docker-for-windows/)の手順に従ってくださ
    い
-   Docker を Mac にインストールするには
    、[こちら](https://docs.docker.com/docker-for-mac/)の手順に従ってください
-   Docker を Linux にインストールするには
    、[こちら](https://docs.docker.com/install/)の手順に従ってください

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義して実行するためのツールです
。[YAML file](https://raw.githubusercontent.com/FIWARE/tutorials.IoT-Sensors/NGSI-LD/docker-compose.yml)
ファイルは、アプリケーションのために必要なサービスを構成するために使用します。つまり、すべてのコンテナ・サービスは
1 つのコマンドで呼び出すことができます。Docker Compose は、デフォルトで Docker for Windows と Docker for Mac
の一部としてインストールされますが、Linux ユーザは[ここ](https://docs.docker.com/compose/install/)に記載されている
手順に従う必要があります。

次のコマンドを使用して、現在の **Docker** バージョンと **Docker Compose** バージョンを確認できます :

```console
docker-compose -v
docker version
```
Docker バージョン 24.0.x 以降と Docker Compose 2.24.x 以上を使用していることを確認し、
必要に応じてアップグレードしてください。

## WSL

シンプルな bash スクリプトを使用してサービスを開始します。Windows ユーザは [を使用して Windows に Linux をインストールする方法](https://learn.microsoft.com/ja-jp/windows/wsl/install)
をダウンロードして、Windows 上の Linux ディストリビューションと同様のコマンドライン機能を提供する必要があります。

<a name="start-up"/>

# 起動

リポジトリ内で提供される bash スクリプトを実行すると、コマンドラインからすべてのサービスを初期化できます。
リポジトリを複製し、以下のコマンドを実行して必要なイメージを作成してください:

```console
git clone https://github.com/FIWARE/tutorials.IoT-Sensors.git
cd tutorials.IoT-Sensors
git checkout NGSI-LD

./services start
```

このコマンドは、起動時に以前の[農場管理の例](https://github.com/FIWARE/tutorials.Context-Providers)
からシードデータをインポートします。

> :information_source: **注:** クリーンアップをやり直したい場合は、次のコマンドを使用して再起動することができます
>
> ```console
> ./services stop
> ```

<a name="communicating-with-iot-devices"/>

# IoT デバイスとの通信

チュートリアルを正しく実行するには、ブラウザのデバイス・モニタのページが表示されていることを確認し、ページをクリックして
cUrl コマンドを入力する前にオーディオを有効にしてください。デバイス・モニタには、Ultralight 2.0 構文を使用してダミー・
デバイスのアレイの現在の状態が表示されます。

#### デバイス・モニタ

デバイス・モニタは次の場所にあります: `http://localhost:3000/device/monitor`

このチュートリアルでは、欠落している IoT Agent コンポーネントの役割を果たし、接続された IoT デバイスへのサウス・バウンド
のコマンドを作成し、ストア内の環境が変化するとノース・バウンドの測定値を受信します。すべてのコマンドは、Ultralight
構文を使用して HTTP POST リクエストとして作成されるため、非常に簡単です。デバイス・モニタのページには、
デバイス自身によって生成されたすべてのノース・バウンドのトラフィックが表示されているので注意が必要です。

<a name="irrigation-system-commands"/>

## Irrigation system (灌漑システム) のコマンド

**Irrigation System** (灌漑システム) のウォーター・スプリンクラーは、アクチュエータの一例です。コマンドに応答できますが、
デバイスは実世界からの測定値を提供しません。

<a name="turn-on-the-irrigation-system"/>

### Irrigation system をオンにする

この例は、実際の IoT Agent がコマンドをアクチュエータに送信する方法を示しています。**Irrigation System** は、
エンドポイント `/iot/water001` を提供し、コマンドを待機しています。

#### 1️⃣ リクエスト:

```console
curl -iX POST 'localhost:3001/iot/water001' \
-H 'Content-Type: text/plain' \
--data-raw 'urn:ngsi-ld:Device:water001@on'
```

#### レスポンス:

```
urn:ngsi-ld:Device:water001@on| on OK
```

リクエストのボディは Ultralight 構文であり、Context Broker に保持されているデバイス (`urn:ngsi-ld:Device:water001`)
の `id` と デバイスで呼び出すコマンド (`on`) の名前で構成されます。

レスポンスは、コマンドとアクションの結果を返します。

デバイス・モニタのページを表示している場合は、ウォーター・スプリンクラーの変化の状態を確認できます。

![](https://fiware.github.io/tutorials.IoT-Sensors/img/water-on.png)

<a name="turn-off-the-irrigation-system"/>

### Irrigation system をオフにする

この例は、irrigation system をオフにする方法を示しています。この場合、デバイスは単一のエンドポイントから
複数のコマンドをリッスンし、ペイロード・ボディを解釈します。

#### 2️⃣ リクエスト:

```console
curl -L -X POST 'localhost:3001/iot/water001' \
-H 'Content-Type: text/plain' \
--data-raw 'urn:ngsi-ld:Device:water001@off'
```

#### レスポンス:

```
urn:ngsi-ld:Device:water001@off| off OK
```

<a name="tractor-commands"/>

## Tractor ( トラクター) のコマンド

Tractor 内にあるシンプルな **FMIS** system を使用すると、Tractor のオペレーターにタスクを送信できます。
また、車両の場所とステータスについて報告することもできます。

測定値は、作業の状態と場所が変更されると、IoT Agent に返されます。

<a name="fmis-system-commands"/>

## FMIS System (FMIS システム) のコマンド

Tractor のダッシュボードの **FMIS System** は、アクチュエータとセンサを組み合わせた例です。

<a name="activate-a-tractor"/>

### Tractor をアクティブ化

この例は、実際の IoT Agent が Ultralight コマンドを **Tractor** FMIS に送信して、アイドル状態からアクティブ状態に
移行する方法を示しています。**Tractor** 自体を備えたユニットは、コマンドを待機するエンドポイント `/iot/tractor001`
をすでに提供しています。

#### 3️⃣ リクエスト:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/tractor001' \
  --data urn:ngsi-ld:Device:tractor001@start
```

リクエストのボディは、Context Broker に保持されているデバイス (`urn:ngsi-ld:Device:tractor001`) の `id` と
デバイスで呼び出すコマンド (`start`) の名前で構成されます。

#### レスポンス:

レスポンスは、コマンドとアクションの結果を返します。

```
urn:ngsi-ld:Device:tractor001@start| start OK
```

Tractor が作動すると、内部 GPS が動きを検出するかどうかに応じて、Tractor の位置と動作が変わります。測定値は
アクティブに報告され、IoT Broker へのリクエストはデバイス・モニタのページで確認できます。

<a name="deactivate-a-tractor"/>

### Tractor を非アクティブ化

この例は、実際の IoT Agent が Ultralight コマンドを **Tractor** FMIS に送信して、車両をアイドル状態に戻す方法を
示しています。**Tractor** はすでにエンドポイント `/iot/tractor001` を提供していて、コマンドを待機しています。

#### 4️⃣ リクエスト:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/tractor001' \
  --data urn:ngsi-ld:Device:tractor001@stop
```

リクエストのボディは、Context Broker に保持されているデバイス (`urn:ngsi-ld:Device:tractor001`) の `id` と
デバイスで呼び出すコマンド (`stop`) の名前で構成されます。

#### レスポンス:

レスポンスは、コマンドとアクションの結果を返します。

```
urn:ngsi-ld:Device:tractor001@stop| stop OK
```

ランプをオフにすると、GPS の位置は変わりません。IoT Agent に送信された最新の Ultralight 測定値
(`s|IDLE|gps|13.36,52.515`) は、デバイス・モニタのページで確認できます。

**Tractor** を再びオンにするには、次のコマンドを繰り返します:

#### 5️⃣ リクエスト:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/tractor001' \
  --data urn:ngsi-ld:Device:tractor001@start
```

#### レスポンス:

```
urn:ngsi-ld:Device:tractor001@start| start OK
```

<a name="filling-station-commands"/>

## Filling Station (充填ステーション) のコマンド

**Filling Station** (充填ステーション)は、飼料をサイロから追加または削除されることをリクエストするコマンドを送信することが
できる電子デバイスです。負荷レベルについても報告できます。実際には、そのようなデバイスは FMIS に接続されて、サイロの飼料が
少なくなりすぎたときに作業員がサイロを充填するようにリクエストしますが、この場合、デバイスはセンサとアクチュエータの
両方として扱われます。

状態が変化すると、測定値が IoT Agent に送信されます。

<a name="remove-hay-from-the-barn"/>

### 納屋から干し草を取り除く

この例は、実際の IoT Agent が Ultralight コマンドを **Filling Station** に送信して、納屋から干し草を取り除く方法を
示しています。**Filling Station** は、すでにエンドポイント `/iot/filling001` を提供していて、コマンドを待機しています。

#### 5️⃣ リクエスト:

```console
curl -iX POST \
  --url 'http://localhost:3001/iot/filling001' \
  --data urn:ngsi-ld:Device:filling001@remove
```

リクエストのボディは、Context Broker に保持されているデバイス (`urn:ngsi-ld:Device:filling001`) の `id` と
デバイスで呼び出すコマンド (`remove`) の名前で構成されます。

#### レスポンス:

レスポンスは、コマンドとアクションの結果を返します。

```
urn:ngsi-ld:Device:filling001@remove| remove OK
```

コマンドが送信されてからしばらくすると、**Filling Station** のレベルが変化します。状態の変化は発生時に IoT Agent
にアクティブに報告され、**Filling Station** の更新された状態はデバイス・モニタのページで確認できます。

農場にある **SoilSensor**, **AnimalCollars** および **TemperatureSensors** はコマンドにレスポンスしていないので、
アクチュエータではありません。ただし、実際の状況を積極的に監視および測定します。 他のコマンドのいずれかが送信された場合、
さまざまなダミー・センサもレスポンスを開始します。

センサによって生成されたノースバウンド HTTP リクエストは、デバイス・モニタのページでも表示できます。

![](https://fiware.github.io/tutorials.IoT-Sensors/img/device-measures.png)

<a name="sending-measures"/>

## 測定値の送信

この例は、デバイス `humidity001` からのリクエストをシミュレートします。

以前にプロビジョニングされたリソース `iot/d` へのリクエストは UltraLight 2.0 形式であり、デバイスの `humidity`
(湿度) を識別し、既知の API キーを渡します。

#### 6️⃣ リクエスト:

```console
curl -L -X POST 'http://localhost:7896/iot/d?k=4jggokgpepnvsb2uv4s40d59ov&i=humidity001' \
-H 'Content-Type: text/plain' \
--data-raw 'h|20'
```

#### レスポンス:

IoT Agent が存在し、適切なエンドポイント (`/iot/d`) でリッスンして測定を処理しない限り、記述された例は機能しません。
ただし、この測定はすべての Ultralight 2.0 によって生成されるリクエストのクラスです。

# 次のステップ

高度な機能を追加することで、アプリケーションに複雑さを加える方法を知りたいですか？ このシリーズの
[他のチュートリアル](https://www.letsfiware.jp/ngsi-ld-tutorials)を読むことで見つけることができます

---

## License

[MIT](LICENSE) © 2020-2024 FIWARE Foundation e.V.
