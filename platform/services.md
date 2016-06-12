# サービス連携

## Outgoing WebHook

モジュールから届いたチャンネルのデータをユーザーのHTTPサーバにPOSTする機能です。

### 設定項目

コントロールパネルでは以下の設定項目があります。

| 項目 | 説明 |
|:----|:-----|
| URL | データを送信するURL |
| Secret | (任意) ペイロードのJSONをHMAC-SHA1で署名する際の秘密鍵 |

### リクエスト

#### HTTPリクエストヘッダ

| ヘッダ | 説明 |
|:----|:-----|
| Content-Type | application/json |
| X-Sakura-Signature | ペイロードとSecretを元に計算した HMAC-SHA1 メッセージ署名 |

設定で `Secret` を指定された場合、`X-Sakura-Signature` ヘッダが付加されます。
この値は `Secret` と HTTPのペイロードの文字列から計算することができます。

この署名をサーバ側で検証することによって、悪意のある第三者から送信されたデータを無視することができます。

```python3
import hmac
import hashlib
x_sakura_signature = hmac.new(secret.encode("utf-8"), payload.encode("utf-8"), hashlib.sha1).hexdigest()
```

#### ペイロード
```
{
    "module": "XXXXXXXXX",
    "type": "channels",
    "datetime": "2016-06-01T12:21:11.628907163Z",
    "payload": {
        "channels": [{
                "channel": 1,
                "type": "i",
                "value": 1
            },
            {
                "channel": 2,
                "type": "b",
                "value": [11, 22, 33, 44, 55, 66, 77, 88]
            }
        ]
    }
}
```



| キー | 説明 |
|:----|:-----|
| module | 送信元モジュールのID |
| type | メッセージの種類 |
| datetime | 送信日時 |
| payload | 内容 |

現在 `type` は `"channels"` のみです。

##### `"channels"` メッセージのアトリビュート

`payload.channels` に送信されたチャンネルの情報が配列で含まれています。

| キー | 説明 |
|:----|:-----|
| payload.channels[].channel | チャンネルID |
| payload.channels[].type | チャンネルの型 |
| payload.channels[].value | 値 |

##### チャンネルの型と値
チャンネルの値の型を示す文字です。

|型|C言語における型|型指定子|`value`の例|
|:----|:--------------|:-------|:---|
|符号あり32bit整数|int32_t|i|-123|
|符号なし32bit整数|uint32_t|I (大文字のアイ)|435|
|符号あり64bit整数|int64_t|l (小文字のエル)|-4425436|
|符号なし64bit整数|uint64_t|L|53465768|
|32bit浮動小数点数|float|f|-3.14|
|64bit浮動小数点数|double|d|435.2344|
|8バイトの配列|uint8_t[8]|b|[11, 22, 33, 44, 55, 66, 77, 88]|



## WebSocket (over SSL)

プラットフォームとWebSocketを使って接続することで、モジュールと双方向にチャンネルデータのやりとりができる機能です。

### 設定項目

コントロールパネルでの設定項目はありません。
コントロールパネルにてWebSocketのサービス連携を作成していただくと、次のようなWebSocketのURLが発行されます。

`wss://secure.sakura.ad.jp/iot-alpha/ws/01234567-abcd-0123-abcd-0123456789ab`

### ペイロード

Outgoing Webhookと同様のJSONで双方向にやりとりを行います。ただし、JSONは改行せず一行で送受信されます。

```
{"module": "XXXXXXXXX","type": "channels","datetime": "2016-06-01T12:21:11.628907163Z","payload": {"channels": [{"channel": 1,"type": "i","value": 1},{"channel": 2,"type": "b","value": [11, 22, 33, 44, 55, 66, 77, 88]}]}}
```

#### `"keepalive"` メッセージ

WebSocketでは接続の維持を確認するために、定期的に以下のようなkeepaliveメッセージをサーバから送信しています。

```
{"type": "keepalive", "datetime": "2016-06-11T06:24:50.643930807Z"}
```
