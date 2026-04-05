# Axios でプロキシを設定する

[![Promo](https://github.com/bright-jp/Rotating-Residential-Proxies/blob/main/50%25%20off%20promo.png)](https://brightdata.jp/proxy-types/residential-proxies) 

この Axios プロキシガイドでは、以下のトピックを扱います。

1. [Axios とプロキシ](#axios-and-proxies)
2. [Axios でプロキシを使う](#using-a-proxy-in-axios)
   - [HTTP/HTTPS プロキシ](#httphttps-proxies)
   - [SOCKS プロキシ](#socks-proxies)
3. [Axios Proxy: 高度なユースケース](#axios-proxy-advanced-use-cases)
   - [プロキシをグローバルに設定する](#setting-a-proxy-globally)
   - [Axios でプロキシ認証に対処する](#dealing-with-proxy-authentication-in-axios)
   - [環境変数でプロキシを設定する](#setting-proxies-via-environment-variables)
   - [ローテーションプロキシを実装する](#implementing-rotating-proxies)
4. [結論](#conclusion)

## Axios とプロキシ

[Axios](https://axios-http.com/) は、JavaScript エコシステムで最も広く使われている HTTP クライアントの 1 つです。Promise ベースで、HTTP リクエストの実行やカスタムヘッダー、設定、cookie の処理を行うための、使いやすく直感的な API を提供します。

Axios のリクエストをプロキシ経由でルーティングすることで、IP アドレスを隠し、ターゲットサーバーがあなたを特定してブロックすることをより難しくできます。

## Axios でプロキシを使う

Axios で HTTP、HTTPS、または SOCKS プロキシを設定してみましょう。`axios` npm パッケージをインストールします。

```bash
npm install axios
```

Node.js では、Axios は [`proxy`](https://github.com/axios/axios#request-config) config を通じて HTTP および HTTPS プロキシをネイティブにサポートしています。したがって、Node.js アプリケーションで Axios と HTTP/HTTPS プロキシを使いたい場合、ここで他に行うことはありません。

一方、HTTP/S 以外のプロキシを使いたい場合は、[Proxy Agents](https://github.com/TooTallNate/proxy-agents) プロジェクトに依存する必要があります。これにより、異なるプロトコルのプロキシと Axios を統合するための `http.Agent` 実装が提供されます。

- HTTP および HTTPS プロキシ: [`https-proxy-agent`](https://github.com/TooTallNate/proxy-agents/blob/main/packages/https-proxy-agent)
- SOCKS、SOCKS5、SOCKS4: [`socks-proxy-agent`](https://github.com/TooTallNate/proxy-agents/blob/main/packages/socks-proxy-agent)
- PAC-\*: [`pac-proxy-agent`](https://github.com/TooTallNate/proxy-agents/blob/main/packages/pac-proxy-agent)

### HTTP/HTTPS プロキシ

HTTP/HTTPS プロキシの URL は次のようになります。

```
"<PROXY_PROTOCOL>://<PROXY_HOST>:<PROXY_PORT>"
```

- `<PROXY_PROTOCOL>` は、HTTP プロキシでは “http”、HTTPS プロキシでは “https” になります。
- `<PROXY_HOST>` は通常、生の IP です。
- `<PROXY_PORT>` は、プロキシサーバーが待ち受けるポートです。

たとえば、HTTP プロキシの URL が次のとおりだとします。

```
"http://47.88.62.42:80"
```

このプロキシは、Axios で次のように設定できます。

```js
axios.get(targetURL, {

    proxy: { 

        protocol: "http", 

        host: "47.88.62.42",

        port: 80

    }

})
```

上記の Axios プロキシアプローチが機能することを確認するには、無料の HTTP または HTTPS プロキシサーバーの URL を取得してください。次の例を試してみましょう。

```
Protocol: HTTP; IP Address: 52.117.157.155; Port: 8002
```

完全なプロキシ URL は `http://52.117.157.155:8002` になります。

プロキシが期待どおりに動作することを確認するには、HTTPBin プロジェクトの [/ip](https://httpbin.io/ip) エンドポイントをターゲットにします。この public API は受信リクエストの IP を返すため、プロキシサーバーの IP を返すはずです。

Node.js スクリプトのスニペットは次のようになります。

```js
import axios from "axios"

async function testProxy() {

    // perform the desired request through the HTTP proxy

const response = await axios.get("https://httpbin.io/ip", {
    proxy: {  
        protocol: "http",  
        host: "52.117.157.155",
        port: 8002
    }
});

    // print the result

    console.log(response.data)

}

testProxy()
```

スクリプトを実行すると、次のようにログ出力されるはずです。

```js
{ "origin": "52.117.157.155" }
```

> **Warning**:\
> スクリプトを実行しても同じ結果にはなりません。無料のプロキシサービスは信頼性が低く、遅く、エラーが発生しやすく、データを貪欲に収集し、寿命も短いためです。

### SOCKS プロキシ

プロキシ config オブジェクトの protocol フィールドに “socks” 文字列を設定しようとすると、次のエラーが発生します。

```js
AssertionError [ERR_ASSERTION]: protocol mismatch

  // ...

 {

  generatedMessage: false,

  code: 'ERR_ASSERTION',

  actual: 'dada:',

  expected: 'http:',

  operator: '=='

}
```

これは、Axios が SOCKS プロキシをネイティブにサポートしていないためです。`socks-proxy-agent` npm ライブラリをプロジェクトの dependencies に追加してください。

```bash
npm install socks-proxy-agent
```

このパッケージを使うと、Axios で HTTP または HTTPS リクエストを行う際に SOCKS プロキシサーバーへ接続できます。

次に、ライブラリから SOCKS プロキシ agent 実装を import します。

```js
const SocksProxyAgent = require("socks-proxy-agent")
```

または、ESM ユーザーの場合は次のとおりです。

```js
import { SocksProxyAgent } from "socks-proxy-agent"
```

SOCKS プロキシの URL が次のとおりだとします。

```
"socks://183.88.74.73:4153"
```

> **Note**:\
> プロキシプロトコルは “socks”、“socks5”、または “socks4” のいずれかです。

これを変数に保存し、`SocksProxyAgent` コンストラクタに渡します。

```js
const proxyURL = "socks://183.88.74.73:4153"

const proxyAgent = new SocksProxyAgent(proxyURL)
```

`SocksProxyAgent()` は、プロキシ URL 経由で HTTP/HTTPS リクエストを実行するための `http.Agent` インスタンスを初期化します。

これで、Axios で SOCKS プロキシを次のように使えます。

```js
axios.get(targetURL, { 

    httpAgent: proxyAgent,     

    httpsAgent: proxyAgent 

})
```

`httpAgent` と `httpsAgent` は、それぞれ HTTP および HTTPS リクエストの実行時に使用するカスタム agent を定義します。つまり、Axios が行う HTTP または HTTPS リクエストは、指定した SOCKS プロキシを経由します。同様に、[`https-proxy-agent`](https://www.npmjs.com/package/https-proxy-agent) npm パッケージを、Axios で HTTP/HTTPS プロキシを設定する別の方法として使用することもできます。

すべてをまとめると、次のようになります。

```js
import axios from "axios"

import { SocksProxyAgent } from "socks-proxy-agent"

async function testProxy() {

    // replace with the URL of your SOCKS proxy 

    const proxyURL = "socks://183.88.74.73:4153"

    // define the HTTP/HTTPS proxy agent

    const proxyAgent = new SocksProxyAgent(proxyURL)

    // perform the request via the SOCKS proxy

    const response = await axios.get("https://httpbin.io/ip", { 

        httpAgent: proxyAgent,     

        httpsAgent: proxyAgent 

    })

    // print the result

    console.log(response.data) // { "origin": "183.88.74.73" }

}

testProxy()
```

その他の例については、[Axios で SOCKS プロキシを設定する方法](https://writech.run/blog/how-to-use-a-socks-proxy-in-axios-6c0355a2e013/) を参照してください。

## Axios Proxy: 高度なユースケース

[![Promo](https://github.com/bright-jp/LinkedIn-Scraper/blob/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.jp/proxy-types/residential-proxies) 

### プロキシをグローバルに設定する

Axios インスタンスで直接指定することで、プロキシをグローバルに設定できます。

```js
const axiosInstance = axios.create({

    proxy: { 

        protocol: "<PROXY_PROTOCOL>", 

        host: "<PROXY_HOST>",

        port: "<PROXY_PORT>" 

    },

    // other configs...

})
```

または、Proxy Agents ユーザーの場合は次のとおりです。

```js
// proxy Agent definition ...

const axiosInstance = axios.create({

    httpAgent: proxyAgent,     

    httpsAgent: proxyAgent 

})
```

Axios が SOCKS プロキシをグローバルに使うよう設定する方法は次のとおりです。

```js
import { SocksProxyAgent } from "socks-proxy-agent";

const proxyURL = "socks://183.88.74.73:4153";

// Create a SOCKS proxy agent
const proxyAgent = new SocksProxyAgent(proxyURL);

// Create an Axios instance with the SOCKS proxy
const axiosInstance = axios.create({
    httpAgent: proxyAgent, // for HTTP requests
    httpsAgent: proxyAgent, // for HTTPS requests
    // other configs...
});
```

これで、`axiosInstance` で行われるすべてのリクエストは、自動的に指定したプロキシを経由するようになります。

### Axios でプロキシ認証に対処する

有料ユーザーのみにプレミアムプロキシへのアクセスを許可するため、プロキシプロバイダーは認証でそれらを保護しています。ユーザー名とパスワードなしで認証付きプロキシに接続しようとすると、[407 Proxy Authentication Required](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/407) エラーになります。

特に、認証付きプロキシの URL 構文は次のとおりです。

```
[<PROTOCOL>://]<USERNAME>:<PASSWORD>@<HOST>[:<PORT>]
```

たとえば、認証付きプロキシに接続する実際の URL は次のようになります。

```
http://admin:lK4w90MEe45YIkOpk@156.127.0.192:8391
```

この場合、プロキシ URL フィールドは次のようになります。

- `<PROTOCOL>:HTTP`
- `<HOST>:156.127.0.192`
- `<PORT>:8391`
- `<USERNAME>:admin`
- `<PASSWORD>:lK4w90MEe45YIkOpk`

Axios でプロキシ認証に対処するには、`proxy` の `authfield` にユーザー名とパスワードを指定します。

```js
axios.get(targetURL, {

    proxy: { 

        protocol: "http", 

        host: "156.127.0.192",

        port: "8381",

        auth: {

            username: "admin",

            password: "lK4w90MEe45YIkOpk"

        }

    }

})
```

一方、Proxy Agents ユーザーの場合、認証に対処する方法は 2 つあります。

1. 認証情報をプロキシ URL に直接追加する:

```js
var proxyAgent = new SocksProxyAgent("http://admin:[email protected]:8391")
```

2. [URL](https://nodejs.org/api/url.html) オブジェクトで `username` および `password` オプションを設定する:

```js
const proxyOpts = new URL("http://156.127.0.192:8391")

proxyOpts.username = "admin"

proxyOpts.password = "lK4w90MEe45YIkOpk"

const proxyAgent = new SocksProxyAgent(proxyOpts)
```

同じアプローチは HttpsProxyAgent でも機能します。

### 環境変数でプロキシを設定する

Axios でプロキシをグローバルに設定する別の方法は、次の環境変数を設定することです。

- `HTTP_PROXY`: HTTP リクエストに使用するプロキシサーバーの URL。
- `HTTPS_PROXY`: HTTPS リクエストに使用するプロキシサーバーの URL。

Linux または macOS では、次のように設定できます。

```bash
export HTTP_PROXY = "[<PROTOCOL>://]<USERNAME>:<PASSWORD>@<HOST>[:<PORT>]"

export HTTPS_PROXY = "[<PROTOCOL>://]<USERNAME>:<PASSWORD>@<HOST>[:<PORT>]"
```

Axios がこれらの環境変数を検出すると、認証用の credentials を含むプロキシ設定をそこから読み取ります。Axios にそれらの環境変数を無視させるには、`proxy` フィールドを `false` に設定してください。また、プロキシを通さないドメインのカンマ区切りリストとして `NO_PROXY` env を定義することもできます。

同じ仕組みは、[cURL でプロキシを使う](https://brightdata.jp/blog/proxy-101/curl-with-proxies) 場合にも機能します。

### ローテーションプロキシを実装する

ターゲットサイトがプロキシの IP アドレスをブロックするのを防ぐには、実行する各リクエストが異なるプロキシサーバーから送信されるようにしてください。

1. それぞれ異なるプロキシへの接続情報を含むオブジェクトのリストを定義する。
2. 各リクエストの前に、プロキシオブジェクトをランダムに選択する。
3. 選択したプロキシを Axios で設定する。

上記のアプローチは、Bright Data が提供する [rotating proxies](https://brightdata.jp/solutions/rotating-proxies) のような、信頼できるプロキシサーバーのプールにアクセスできることを前提としています。

## 結論

Bright Data は世界最高のプロキシサーバーを管理しており、Fortune 500 企業や 20,000 を超える顧客にサービスを提供しています。その世界規模のプロキシネットワークには、以下が含まれます。

*   [Datacenter proxies](https://brightdata.jp/proxy-types/datacenter-proxies) – 770,000 を超える datacenter IP。
*   [Residential proxies](https://brightdata.jp/proxy-types/residential-proxies) – 195 か国以上にまたがる 72M を超える residential IP。
*   [ISP proxies](https://brightdata.jp/proxy-types/isp-proxies) – 700,000 を超える ISP IP。
*   [Mobile proxies](https://brightdata.jp/proxy-types/mobile-proxies) – 7M を超える mobile IP。

今すぐ [無料の Bright Data アカウントを作成](https://brightdata.jp/#popup-155639) して、当社のプロキシサーバーをお試しください。