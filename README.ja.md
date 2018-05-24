![FIWARE Banner](https://fiware.github.io/tutorials.IoT-Sensors//img/fiware.png)

[![NGSI v2](https://img.shields.io/badge/Ultralight-2.0-pink.svg)](http://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)

このチュートリアルでは、IoT デバイスの概要と、制約のあるデバイス(constrained devices)用の [UltraLight 2.0](http://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual) プロトコルの使用方法について説明します。このチュートリアルでは、一連のダミー IoT デバイスを紹介します。これらのデバイスは、ブラウザ内に表示され、ユーザがそれらとやりとりすることを可能にします。IoT デバイスを実際の IoT Agent を介して Orion Context Broker に接続する前に、このチュートリアルで定義されているすべての用語と概念を完全に理解する必要があります。

このチュートリアルでは、全体で [cUrl](https://ec.haxx.se/) コマンドを使用していますが、[Postman documentation](http://fiware.github.io/tutorials.Getting-Started/) も利用できます。

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/32975e01a2c250698149)

# コンテンツ

- [IoT デバイスとは何ですか？](#what-are-iot-devices)
- [Ultralight 2.0 とは何ですか？](#what-is-ultralight-20)
  * [サウス・バウンドのトラフィック (コマンド)](#southbound-traffic-commands)
    + [HTTP POST を使用したプッシュコマンド](#push-command-using-http-post)
  * [ノース・バウンドのトラフィック(測定)](#northbound-traffic-measurements)
    + [HTTP GET を使用した測定](#measurement-using-http-get)
    + [HTTP POST を使用した測定](#measurement-using-http-post)
- [アーキテクチャ](#architecture)
- [前提条件](#prerequisites)
  * [Docker](#docker)
  * [Cygwin](#cygwin)
- [起動](#start-up)
- [IoT デバイスとの通信](#communicating-with-iot-devices)
  * [ベル・コマンド](#bell-commands)
    + [ベルを鳴らす](#ring-a-bell)
  * [スマート・ランプのコマンド](#smart-lamp-commands)
    + [スマート・ランプのスイッチを入れる](#switch-on-a-smart-lamp)
    + [スマート・ランプのスイッチを切る](#switch-off-a-smart-lamp)
  * [スマート・ドアのコマンド](#smart-door-commands)
    + [ドアのロックを解除する](#unlock-a-door)
    + [ドアを開く](#open-a-door)
    + [ドアを閉じる](#close-a-door)
    + [ドアをロックする](#lock-a-door)
- [次のステップ](#next-steps)

<a name="what-are-iot-devices"></a>
# IoTデバイスとは何ですか？

[Internet of Things](https://www.linux.com/news/who-needs-internet-things) は、ネットワークに接続し、データを交換できる物理デバイスのネットワークです。それぞれの"モノ"または"スマートデバイス"は、センサまたはアクチュエータとして機能する電子機器およびソフトウェアを内蔵したガジェットです。センサは、それらの周囲の現実の状態を報告することができます。アクチュエータは、制御信号に応答することによって、システムの状態を変更する役割を担います。

各デバイスは、組み込みコンピューティング・システムを通じて一意に識別可能ですが、既存のインターネットのインフラストラクチャ内で相互運用することができます。

FIWARE は、コンテキスト情報を管理するシステムです。IoT に基づくスマートなソリューションのために、コンテキストは取り付けられたIoT デバイスのアレイによって提供されます。各 IoT デバイスは現実世界に存在する物理的オブジェクトであるため、最終的にコンテキスト内の一意のエンティティとして表現されます。

IoT デバイスは、単純なものから複雑なものまで様々です。このチュートリアルで使用する IoT デバイスの例をいくつか示します :

* **スマート・ドア**(Smart Door)は、リモートでロックまたはアンロックするコマンドを送信できる電子ドアです。現在の状態(OPEN, CLOSED または LOCKED)を報告することもできます
* **ベル**(Bell)は、アクティブにするコマンドを送信すると、短期間ベルを鳴らすことができます
* **モーション・センサ**(Motion Sensor)は、最後にリセットされてから、通過した人の数を返すためにクエリすることができます
* **スマート・ランプ**(Smart Lamp)は、リモートでオンまたはオフに切り替えることができます。また、現在の状態(ONまたはOFF)を報告することもできます。スイッチがオンになると、デバイス内のモーション・センサが光が必要かどうかを確認し、近くに人がいない場合は減光します。さらに、この装置は、電球の現在の光度を報告することができます

このように、ベルは純粋なアクチュエータの例です。ベルは与えられたコマンドにしか反応しません。一方、 モーション・センサは純粋なセンサの一例であり、世界の状況についてのみ報告します。他の2つのデバイスは、コマンドに応答して状態を意味のある方法で報告することができます。

各デバイス内に保持されている状態情報は、Context Broker 内で最終的に認識されるため、以下の図に定義されています :

![](https://fiware.github.io/tutorials.IoT-Sensors/img/entities.png)

<a name="what-is-ultralight-20"></a>
#  Ultralight 2.0 とは何ですか？

[UltraLight 2.0](http://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual) は、帯域幅とデバイスのメモリ・リソースが制限されている、制約のあるデバイスや通信用の軽量テキストベースのプロトコルです。測定要求のペイロードは、パイプ `|` 文字で区切られたキーと値のペアのリストです。

例えば

```
<key>|<value>|<key>|<value>|<key>|<value> etc..
```

例えば、次のようなペイロード :

```
t|15|k|abc
```

2つの属性が含まれています。1つは、値 "15" を持つ名前 "t" の属性、もう1つは、値 "abc" を持つ名前 "k" の属性が送信されます。Ultralight 2.0 では値(Value)のタイプはありません。すべてが文字列として扱われます。

Ultralight 2.0 は、デバイスとサーバ間で共有する測定値とコマンドを記述するペイロードを定義しますが、単一のトランスポート・プロトコルは指定しません。代わりに、さまざまなシナリオで異なるトランスポート・プロトコル・バインディング(HTTP, MQTT および AMQP など)を使用できます。このチュートリアルでは、HTTP をトランスポート・プロトコルとして使用します。

<a name="southbound-traffic-commands"></a>
## サウス・バウンドのトラフィック (コマンド)

Context Broker から生成され、IoT Agent を介して IoT デバイスに向けて下向きに渡された HTTP リクエストを、サウス・バウンド・トラフィックと呼びます。サウス・バウンドのトラフィックは、実世界の状態を動作によって変更するアクチュエータ・デバイスに対する**コマンド**で構成されています。例えば、ランプの状態を `ON` に変更して実際のランプを点灯させるコマンドがあります。これは、近くの他センサの読み取り値を変更する可能性があります。
the readings of other sensors nearby.

<a name="push-command-using-http-post"></a>
###  HTTP POST を使用したプッシュコマンド

IoT Agent と IoT デバイスとの間のサウス・バウンド通信を設定することを、プロビジョニングと呼びます。これにより、IoT Agent が各 IoT デバイスに接続できるだけの十分な情報を保持することが保証されます。つまり、コマンドを送信する場所とサポートされているコマンドを認識します。IoT Agent は、デバイスにコマンドを送信するために、デバイスが提供するエンドポイントに POST リクエストを送信します。POST リクエストの本体にはコマンドが保持されます。

Ultralight コマンドのペイロードの形式は次のとおりです :

```
<device name>@<command>|<param|<param>
```

ここで、`<device_name>` は、Context Broker に保持されているエンティティ `id`です。`<command>` は、サポートされているコマンドの1つであり、追加の必須値は後続のパラメータで渡されます。

```
urn:ngsi-ld:Robot:001@turn|left|30
```

*"私はContext Broker内で `id="urn:ngsi-ld:Robot:001"` として知られています。このエンドポイントでリッスンしているデバイスが `turn` コマンドを実行するようにしたいと思います。 私は、操作を実行できるようにデバイスに必要なパラメータ `left` and '`30` (degrees) を提供しました。"* をデバイスに通知します。

IoT Agent に対する定義されたノース・バウンドのレスポンスは、次のとおりです :

```
urn:ngsi-ld:Robot:001@turn|Turn ok
```

*"Context Broker 内で `id="urn:ngsi-ld:Robot:001"` というエンティティからのリクエストに対応しました。 私が行ったコマンドは、`turn` コマンドでした。 結果は `Turn ok` になりました。"* と言っています。

このように、サウス・バウンドのコマンドは、やりとりで使用される `id` を定義し、同じデータも返されるため、すべてのレスポンスは常に Context Broker 内に保持されている適切なエンティティに関連付けることができます。

プッシュ・コマンドは、デバイスがサウス・バウンドのトラフィックを受信するための別個のエンドポイントを提供できる場合にのみ使用できます。デバイスからすべてのインタラクションが開始されたときに代替ポーリング・メカニズムを使用できますが、これはこのチュートリアルの範囲を超えています。

<a name="northbound-traffic-measurements"></a>
## ノース・バウンドのトラフィック(測定)

IoT デバイスから生成され、IoT Agent を介して Context Broker に向けて上向きに戻されたリクエストは、ノース・バウンドのトラフィックと呼ばれます。ノース・バウンドのトラフィックは、センサ・デバイスによって行われた測定からなり、現実世界の状態をシステムのコンテキスト・データに中継します。例えば、湿度センサからの測定値を Context Broker に戻して、エンティティの水分レベルが変化したことを示すことができます。そのような変更を通知されるようにサブスクリプションを作成し、さらなるアクションを実行することができます。例えば、スプリンクラーをオンにするなどのアクションです。

<a name="measurement-using-http-get"></a>
### HTTP GET を使用した測定

デバイスは、次のクエリ・パラメータと共に "well-known" エンドポイント(パス `/iot/d`)への HTTP GET リクエストを使用して、IoT プラットフォームに新しい測定値を報告できます :

* `i` (device ID): デバイス ID。API キーで一意
* `k` (API Key): デバイスが登録されているサービスの API のキー
* `t` (timestamp): 測定のタイムスタンプ。自動の IoT Agent タイムスタンプを無効にします (オプション)
* `d` (Data): Ultralight 2.0 ペイロード

`i` と `k` パラメータは必須です。

例えば、リクエスト :

```
<iot-agent>/iot/d?i=motion001&d=c|12
```

これは、デバイス `id=motion001` が、値 `12` で実測値 `c` を行ったことを IoT Agent に知らせることを望むことを示しています。これは最終的に Context Broker に渡されます。

<a name="measurement-using-http-post"></a>
### HTTP POST を使用した測定

HTTP POST も使用できます。再び、パスは、`/iot/d` になりますが、この場合には、`d` (データ)は不要です。測定値のキーと値のペアがリクエストの本体として渡されます。`i` と `k` のクエリ・パラメータは依然として必須です :

* `i` (device ID): デバイス ID。API キーで一意
* `k` (API Key): デバイスが登録されているサービスの API のキー
* `t` (timestamp): 測定のタイムスタンプ。自動の IoT Agent タイムスタンプを無効にします (オプション)

`i` と `k` パラメータは必須です。


#### デバイス・モニタ

このチュートリアルでは、一連のダミーの IoT デバイスを作成しました。このデバイスは、最終的に Context Broker に接続されます。各デバイスの状態は、次の UltraLight デバイス・モニタの Web ページで確認できます : `http://localhost:3000/device/monitor`

![FIWARE Monitor](https://fiware.github.io/tutorials.IoT-Sensors/img/device-monitor.png)


<a name="architecture"></a>
# アーキテクチャ

デモアプリケーションは、ダミーの IoT デバイスのセットとして機能する単一のカスタム・コンポーネントのみを使用します。すべての IoT デバイスは、HTTP 上で動作する [UltraLight 2.0](http://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual) プロトコルを使用します。すべての対話は HTTP リクエストによって開始されるため、エンティティはコンテナ化され、公開されたポートから実行されます。

![](https://fiware.github.io/tutorials.IoT-Sensors/img/architecture.png)

実際のスマート・ソリューションを通してメッセージを説明するとき、このチュートリアルでは使用されていない2つのコンポーネントを参照しますが、後でシステムを完了するためには必要になります。

* Orion Context Broker サーバは、スマート・ソリューションのコンテキスト・データを保持するために使用されます。ご存知のように、Context Broker とのやりとりは、すべて [NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2) を使用して行う必要があります
* IoT Agent は 、Context Broker からの [NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2) リクエストを、IoT デバイス自体が使用できるプロトコル(UltraLight 2.0 など)に変換するミドルウェア・コンポーネントとして機能します

したがって、サンプルのデバイス・プロトコルを最初に理解し、次に IoT Agent ミドルウェアの目的を理解するために、メッセージがシステムをどのように通過するかを理解する必要があります。このチュートリアルでは、あなたが IoT Agent の役割を果たし、デバイスにコマンドを送り、デバイスからの測定値を受信します。


<a name="prerequisites"></a>
# 前提条件

<a name="docker"></a>
## Docker

物事を単純にするために、両方のコンポーネントが [Docker](https://www.docker.com) を使用して実行されます。**Docker** は、さまざまコンポーネントをそれぞれの環境に分離することを可能にするコンテナ・テクノロジです。

* Docker を Windows にインストールするには、[こちら](https://docs.docker.com/docker-for-windows/)の手順に従ってください
* Docker を Mac にインストールするには、[こちら](https://docs.docker.com/docker-for-mac/)の手順に従ってください
* Docker を Linux にインストールするには、[こちら](https://docs.docker.com/install/)の手順に従ってください

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義して実行するためのツールです。[YAML file](htt
ps://raw.githubusercontent.com/Fiware/tutorials.Getting-Started/master/docker-compose.yml) ファイルは、アプリケーションのために必要なサービスを設定する使用されています。つまり、すべてのコンテナ・サービスは1つのコマンドで呼び出すことができます。Docker Compose は、デフォルトで Docker for Windows とD ocker for Mac の一部としてインストールされますが、Linux ユーザは[ここ](https://docs.docker.com/compose/install/)に記載されている手順に従う必要があります。

<a name="cygwin"></a>
## Cygwin 

シンプルな bash スクリプトを使用してサービスを開始します。Windows ユーザは [cygwin](http://www.cygwin.com/) をダウンロードして、Windows 上の Linux ディストリビューションと同様のコマンドライン機能を提供する必要があります。

<a name="start-up"></a>
# 起動

リポジトリ内で提供される bash スクリプトを実行すると、コマンドラインからすべてのサービスを初期化できます :

```console
./services create; ./services start;
```

このコマンドは、起動時に以前の[在庫管理の例](https://github.com/Fiware/tutorials.Context-Providers)からシードデータをインポートします。

>:information_source: **注** : クリーンアップをやり直したい場合は、次のコマンドを使用して再起動することができます :
>
>```console
>./services stop
>``` 
>

<a name="communicating-with-iot-devices"></a>
#  IoT デバイスとの通信

チュートリアルを正しく実行するには、ブラウザのデバイス・モニタのページが表示されていることを確認し、ページをクリックして cUrl コマンドを入力する前にオーディオを有効にしてください。デバイス・モニタには、Ultralight 2.0 構文を使用してダミー・デバイスのアレイの現在の状態が表示されます。

#### デバイス・モニタ

デバイス・モニタは次の場所にあります : `http://localhost:3000/device/monitor`

このチュートリアルでは、欠落している IoT Agent コンポーネントの役割を果たし、接続された IoT デバイスへのサウス・バウンドのコマンドを作成し、ストア内の環境が変化するとノース・バウンドの測定値を受信します。すべてのコマンドは、Ultralight 構文を使用して HTTP POST リクエストとして作成されるため、非常に簡単です。デバイス・モニタのページには、デバイス自身によって生成されたすべてのノース・バウンドのトラフィックが表示されているので注意が必要です。

<a name="bell-commands"></a>
## ベル・コマンド

**ベル**(Bell)は、アクチュエータの一例です。これはコマンドに応答することができますが、デバイスは実際の測定値を供給しません。

<a name="ring-a-bell"></a>
### ベルを鳴らす

IoT broker がどのようにアクチュエータにコマンドを送信するかを示しています。**Bell**(ベル)は、エンドポイント `/iot/bell001` を提供していて、コマンドをリッスンしています。

#### リクエスト :

```console
curl --request POST \
  --url 'http://localhost:3001/iot/bell001' \
  --data urn:ngsi-ld:Bell:001@ring
```

#### レスポンス :

```
urn:ngsi-ld:Bell:001@ring| ring OK
```

リクエストの本体は、Ultralight の構文であり、Context Broker に保持されているデバイス(`urn:ngsi-ld:Bell:001`)の `id` と、デバイス上で呼び出すコマンド(`ring`)の名前で構成されています。

レスポンスは、アクションのコマンドと結果を返します。

デバイス・モニタのページが表示されている場合は、ベルの状態変更を確認できます。

![](https://fiware.github.io/tutorials.IoT-Sensors//img/bell-ring.gif)


<a name="smart-lamp-commands"></a>
## スマート・ランプのコマンド

**スマート・ランプ**(Smart Lamp)はリモートでオンとオフを切り替えることができます。光度も登録されます。その中にはモーション・センサが含まれており、動きが検出されなければ、時間の経過と共にゆっくりと薄暗くなります。

状態および/または光度が変化すると、IoT Agent に測定値が送信されます。

<a name="switch-on-a-smart-lamp"></a>
### スマート・ランプのスイッチを入れる

この例は、IoT broker が**スマート・ランプ**に Ultralight  コマンドを送信してスイッチをオンにする方法を示しています。**スマート・ランプ**は、エンドポイント `/iot/lamp001` を提供していて、コマンドをリッスンしています。

#### リクエスト :

```console
curl --request POST \
  --url 'http://localhost:3001/iot/lamp001' \
  --data urn:ngsi-ld:Lamp:001@on
```
リクエストの本体は、Context Brokerに保持されているデバイス(`urn:ngsi-ld:Lamp:001`)の `id` と、デバイス上で呼び出すコマンド(`on`)の名前で構成されています。

#### レスポンス :

レスポンスは、アクションのコマンドと結果を返します。
```
urn:ngsi-ld:Lamp:001@on| on OK
```

ランプが点灯(on)すると、光度レベルは、内部のモーション・センサが動きを検出するかどうかに依存して変化します。測定値は積極的に報告され、IoT Broker へのリクエストはデバイス・モニタのページに表示されます。


<a name="switch-off-a-smart-lamp"></a>
### スマート・ランプのスイッチを切る

この例は、 IoT broker が**スマート・ランプ**に Ultralight コマンドを送信してスイッチをオフにする方法を示しています。**スマート・ランプ**はエンドポイント `/iot/lamp001` を提供していて、コマンドをリッスンしています。

#### リクエスト :

```console
curl --request POST \
  --url 'http://localhost:3001/iot/lamp001' \
  --data urn:ngsi-ld:Lamp:001@off
```
リクエストの本体は、Context Broker に保持されているデバイス(`urn:ngsi-ld:Lamp:001`)の `id` と、デバイス上で呼び出すコマンド(`off`)の名前で構成されています。

#### レスポンス :

レスポンスは、アクションのコマンドと結果を返します。

```
urn:ngsi-ld:Lamp:001@off| off OK
```

ランプが消灯(off)すると、光度は変化しません。IoT broker に送信された最新の Ultralight 測定値(`s|OFF|l|0`)は、デバイス・モニタのページで確認できます。

スマート・ランプを再びオンにするには、次のコマンドを繰り返します :

#### リクエスト :

```console
curl --request POST \
  --url 'http://localhost:3001/iot/lamp001' \
  --data urn:ngsi-ld:Lamp:001@on
```

#### レスポンス :

```
urn:ngsi-ld:Lamp:001@on| on OK
```


<a name="smart-door-commands"></a>
## スマート・ドアのコマンド
**スマート・ドア**(Smart Door)は、リモートでロックまたはアンロックするコマンドを送信できる電子ドアです。また、現在の状態(`OPEN`, `CLOSED` または `LOCKED`)を報告することもできます。

状態が変化すると、測定値が IoT Agent に送信されます。

<a name="unlock-a-door"></a>
### ドアのロックを解除する

この例では、 IoT broker が**スマート・ドア**に Ultralight コマンドを送信してドアのロックを解除する方法を示します。**スマート・ドア**はエンドポイント `/iot/door001` を提供していて、コマンドをリッスンしています。

#### リクエスト :

```console
curl --request POST \
  --url 'http://localhost:3001/iot/door001' \
  --data urn:ngsi-ld:Door:001@unlock
```
リクエストの本文は、Context Broker に保持されているデバイス(`urn:ngsi-ld:Door:001`) の `id`と、デバイス上で呼び出すコマンド(`unlock`)の名前で構成されています。

#### レスポンス :

レスポンスは、アクションのコマンドと結果を返します。

```
urn:ngsi-ld:Door:001@unlock| unlock OK
```

**スマート・ドア**のロックが解除されれば、顧客が入ると自動的に開閉ます。状態の変化は積極的に  IoT broker に報告され、**スマート・ドア**の状態はデバイス・モニタのページに表示されます。

店内のモーション・センサはアクチュエータではなく、コマンドに応答しませんが、通過する顧客の数を積極的に測定します。ドアがロックされていない場合、モーション・センサは動きを検出し、Ultralight 測定値を  IoT broker にバックアップします。

**モーション・センサ**によって生成された、ノース・バウンドの HTTP リクエストは、デバイス・モニタのページでも表示できます。

![](https://fiware.github.io/tutorials.IoT-Sensors//img/door-open.gif)


<a name="open-a-door"></a>
### ドアを開く

この例では、 IoT broker がドアを開くために**スマート・ドア**にコマンドを送信する方法を示します。**スマート・ドア**はエンドポイント `/iot/door001` を提供していて、コマンドをリッスンしています。

#### リクエスト :

```console
curl --request POST \
  --url 'http://localhost:3001/iot/door001' \
  --data urn:ngsi-ld:Door:001@open
```
リクエストの本体は、Context Broker に保持されているデバイス(`urn:ngsi-ld:Door:001`) の `id` と、デバイス上で呼び出すコマンド(`open`)の名前で構成されています。

#### レスポンス :

レスポンスは、アクションのコマンドと結果を返します。

```
urn:ngsi-ld:Door:001@open| open OK
```

**スマート・ドア**の状態は、デバイス・モニタのページで確認できます。顧客が入店すれば、**モーション・センサ**は動きを拾い、測定値を IoT broker に送ることができます。

**スマート・ドア**と**モーション・センサ**によって生成された、ノース・バウンドの HTTP リクエストは、デバイス・モニタのページでも表示できます。

<a name="close-a-door"></a>
### ドアを閉じる

この例は、IoT broker がドアを閉じるために**スマート・ドア**にコマンドを送信する方法を示しています。**スマート・ドア**はエンドポイント `/iot/door001` を提供していて、コマンドをリッスンしています。

#### リクエスト :

```console
curl --request POST \
  --url 'http://localhost:3001/iot/door001' \
  --data urn:ngsi-ld:Door:001@close
```
リクエストの本体は、Context Brokerに保持されているデバイス(`urn:ngsi-ld:Door:001`) の `id` と、デバイス上で呼び出すコマンド(`close`)の名前で構成されています。

#### レスポンス :

レスポンスは、アクションのコマンドと結果を返します。

```
urn:ngsi-ld:Door:001@close| cloes OK
```

ドアは現在ロックされていないので、顧客は引き続きドアに入り、ドアをサイドオープンします。動きが検出されると、**モーション・センサ**は IoT broker に測定値を送信します。

**モーション・センサ**によって生成された、ノース・バウンドの HTTP リクエストは、デバイス・モニタの
ページでも表示できます。

<a name="lock-a-door"></a>
### ドアをロックする

この例では、ドアをロックするために IoT broker が Ultralight コマンドを**スマート・ドア**に送信する方法を示します。**スマート・ドア**はエンドポイント `/iot/door001` を提供していて、コマンドをリッスンしています。

#### リクエスト :

```console
curl --request POST \
  --url 'http://localhost:3001/iot/door001' \
  --data urn:ngsi-ld:Door:001@lock
```

リクエストの本体は、Context Broker に保持されているデバイス(`urn:ngsi-ld:Door:001`)の `id` と、デバイス上で呼び出すコマンド(`lock`)の名前で構成されています。

#### レスポンス :

レスポンスは、アクションのコマンドと結果を返します。

```
urn:ngsi-ld:Door:001@close| close OK
```

ドアがロックされると、それ以上、顧客は入店できません。**モーション・センサ**はさらなる動きが検出されなかったと報告します。**スマート・ドア**を手動で開けることができず、**スマート・ランプ**はゆっくりと周囲の照明レベルに戻ります。

スマート・ランプによって生成された、ノース・バウンドの HTTP リクエスト は、デバイス・モニタのページで表示できます。

![](https://fiware.github.io/tutorials.IoT-Sensors//img/door-lock.gif)


<a name="next-steps"></a>
# 次のステップ

高度な機能を追加することで、アプリケーションに機能を追加する方法を知りたいですか？ このシリーズの他のチュートリアルを読むことで見つけることができます :

&nbsp; 101. [Getting Started](https://github.com/Fiware/tutorials.Getting-Started)<br/>
&nbsp; 102. [Entity Relationships](https://github.com/Fiware/tutorials.Entity-Relationships/)<br/>
&nbsp; 103. [CRUD Operations](https://github.com/Fiware/tutorials.CRUD-Operations/)<br/>
&nbsp; 104. [Context Providers](https://github.com/Fiware/tutorials.Context-Providers/)<br/>
&nbsp; 105. [Altering the Context Programmatically](https://github.com/Fiware/tutorials.Accessing-Context/)<br/> 
&nbsp; 106. [Subscribing to Changes in Context](https://github.com/Fiware/tutorials.Subscriptions/)<br/>

&nbsp; 201. [Introduction to IoT Sensors](https://github.com/Fiware/tutorials.IoT-Sensors/)<br/>






