# Axiosでプロキシを設定する

[![Promo](https://github.com/bright-jp/Rotating-Residential-Proxies/blob/main/50%25%20off%20promo.png)](https://brightdata.jp/proxy-types/residential-proxies) 

このAxiosプロキシガイドでは、以下のトピックを扱います。

1. [Axiosとプロキシ](#axios-and-proxies)
2. [Axiosでプロキシを使用する](#using-a-proxy-in-axios)
   - [HTTP/HTTPSプロキシ](#httphttps-proxies)
   - [SOCKSプロキシ](#socks-proxies)
3. [Axiosプロキシ：高度なユースケース](#axios-proxy-advanced-use-cases)
   - [プロキシをグローバルに設定する](#setting-a-proxy-globally)
   - [Axiosでプロキシ認証に対処する](#dealing-with-proxy-authentication-in-axios)
   - [環境変数でプロキシを設定する](#setting-proxies-via-environment-variables)
   - [ローテーティングプロキシを実装する](#implementing-rotating-proxies)
4. [結論](#conclusion)

## Axiosとプロキシ

[Axios](https://axios-http.com/) は、JavaScriptエコシステムで最も広く使われているHTTPクライアントの1つです。HTTPリクエストの実行や、カスタムヘッダー、設定、Cookieの取り扱いに向けて、Promiseベースで使いやすく直感的なAPIを提供します。

Axiosのリクエストをプロキシ経由でルーティングすることで、IPアドレスを隠蔽でき、ターゲットサーバーがあなたを識別してブロックすることがより難しくなります。

## Axiosでプロキシを使用する

AxiosでHTTP、HTTPS、またはSOCKSプロキシを設定しましょう。`axios` npmパッケージをインストールします。

```bash
npm install axios
```

Node.jsでは、Axiosは[`proxy`](https://github.com/axios/axios#request-config) 設定によりHTTPおよびHTTPSプロキシをネイティブにサポートしています。そのため、Node.jsアプリケーションでAxiosにHTTP/HTTPSプロキシを使いたい場合、ここで他に行うことはありません。

一方で、HTTP/S以外のプロキシを使用したい場合は、[Proxy Agents](https://github.com/TooTallNate/proxy-agents) プロジェクトに頼る必要があります。これは、さまざまなプロトコルのプロキシをAxiosに統合するための `http.Agent` 実装を提供します。

- HTTPおよびHTTPSプロキシ：[`https-proxy-agent`](https://github.com/TooTallNate/proxy-agents/blob/main/packages/https-proxy-agent)
- SOCKS、SOCKS5、SOCKS4：[`socks-proxy-agent`](https://github.com/TooTallNate/proxy-agents/blob/main/packages/socks-proxy-agent)
- PAC-\*：[`pac-proxy-agent`](https://github.com/TooTallNate/proxy-agents/blob/main/packages/pac-proxy-agent)

### HTTP/HTTPSプロキシ

HTTP/HTTPSプロキシのURLは次のような形式になります。

```
"<PROXY_PROTOCOL>://<PROXY_HOST>:<PROXY_PORT>"
```

- `<PROXY_PROTOCOL>` は、HTTPプロキシの場合は “http”、HTTPSプロキシの場合は “https” になります。
- `<PROXY_HOST>` は一般的に生のIPです。
- `<PROXY_PORT>` はプロキシサーバーが待ち受けるポートです。

たとえば、あなたのHTTPプロキシのURLが次のとおりだとします。

```
"http://47.88.62.42:80"
```

このプロキシは、次のようにAxiosで設定できます。

```js
axios.get(targetURL, {

    proxy: { 

        protocol: "http", 

        host: "47.88.62.42",

        port: 80

    }

})
```

上記のAxiosプロキシの方法が動作することを検証するには、無料のHTTPまたはHTTPSプロキシサーバーのURLを取得してください。次の例を試してください。

```
Protocol: HTTP; IP Address: 52.117.157.155; Port: 8002
```

プロキシURL全体は `http://52.117.157.155:8002` になります。

プロキシが期待どおりに動作することを確認するには、HTTPBinプロジェクトの [/ip](https://httpbin.io/ip) エンドポイントをターゲットにしてください。この公開APIは受信リクエストのIPを返すため、プロキシサーバーのIPが返るはずです。

Node.jsスクリプトのスニペットは次のとおりです。

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
> スクリプトを実行しても同じ結果にはなりません。無料プロキシサービスは信頼性が低く、遅く、エラーが起きやすく、データを貪欲に収集し、寿命も短いからです。

### SOCKSプロキシ

プロキシ設定オブジェクトのprotocolフィールドに “socks” 文字列を設定しようとすると、次のエラーが発生します。

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

これは、AxiosがSOCKSプロキシをネイティブにサポートしていないためです。`socks-proxy-agent` npmライブラリをプロジェクトの依存関係に追加してください。

```bash
npm install socks-proxy-agent
```

このパッケージにより、AxiosでHTTPまたはHTTPSリクエストを行いながら、SOCKSプロキシサーバーに接続できます。

次に、ライブラリからSOCKSプロキシagent実装をインポートします。

```js
const SocksProxyAgent = require("socks-proxy-agent")
```

または、ESMユーザーの場合は次のとおりです。

```js
import { SocksProxyAgent } from "socks-proxy-agent"
```

あなたのSOCKSプロキシのURLが次のとおりだとします。

```
"socks://183.88.74.73:4153"
```

> **Note**:\
> プロキシプロトコルは “socks”、 “socks5”、または “socks4” のいずれかにできます。

これを変数に格納し、`SocksProxyAgent` コンストラクタに渡します。

```js
const proxyURL = "socks://183.88.74.73:4153"

const proxyAgent = new SocksProxyAgent(proxyURL)
```

`SocksProxyAgent()` は、プロキシURL経由でHTTP/HTTPSリクエストを実行するための `http.Agent` インスタンスを初期化します。

これで、次のようにAxiosでSOCKSプロキシを使用できます。

```js
axios.get(targetURL, { 

    httpAgent: proxyAgent,     

    httpsAgent: proxyAgent 

})
```

`httpAgent` と `httpsAgent` は、それぞれHTTPおよびHTTPSリクエストを実行する際に使用するカスタムagentを定義します。言い換えると、Axiosが行うHTTPまたはHTTPSリクエストは、指定したSOCKSプロキシを経由します。同様に、AxiosでHTTP/HTTPSプロキシを設定する別の方法として、[`https-proxy-agent`](https://www.npmjs.com/package/https-proxy-agent) npmパッケージを使用することもできます。

すべてをまとめると次のようになります。

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

他の例については、[AxiosでSOCKSプロキシを設定する方法](https://writech.run/blog/how-to-use-a-socks-proxy-in-axios-6c0355a2e013/) のリンクをご参照ください。

## Axiosプロキシ：高度なユースケース

[![Promo](https://github.com/bright-jp/LinkedIn-Scraper/blob/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.jp/proxy-types/residential-proxies) 

### プロキシをグローバルに設定する

Axiosインスタンスに直接指定することで、プロキシをグローバルに設定できます。

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

または、Proxy Agentsユーザーであれば次のとおりです。

```js
// proxy Agent definition ...

const axiosInstance = axios.create({

    httpAgent: proxyAgent,     

    httpsAgent: proxyAgent 

})
```

SOCKSプロキシをグローバルに使用するようにAxiosを設定する方法は次のとおりです。

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

`axiosInstance` で行われるすべてのリクエストは、以後自動的に指定のプロキシを経由します。

### Axiosでプロキシ認証に対処する

プレミアムプロキシへのアクセスを有料ユーザーのみに許可するため、プロキシプロバイダーは認証で保護しています。ユーザー名とパスワードなしで認証付きプロキシに接続しようとすると、[407 Proxy Authentication Required](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/407) エラーになります。

特に、認証付きプロキシのURL構文は次のとおりです。

```
[<PROTOCOL>://]<USERNAME>:<PASSWORD>@<HOST>[:<PORT>]
```

たとえば、認証付きプロキシに接続する実世界のURLは次のようになります。

```
http://admin:lK4w90MEe45YIkOpk@156.127.0.192:8391
```

この場合、プロキシURLフィールドは次のとおりです。

- `<PROTOCOL>:HTTP`
- `<HOST>:156.127.0.192`
- `<PORT>:8391`
- `<USERNAME>:admin`
- `<PASSWORD>:lK4w90MEe45YIkOpk`

Axiosでプロキシ認証に対処するには、`proxy` の `authfield` にユーザー名とパスワードを指定します。

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

一方、Proxy Agentsユーザーの場合は、認証に対処する方法が2つあります。

1. 認証情報をプロキシURLに直接追加します。

```js
var proxyAgent = new SocksProxyAgent("http://admin:[email protected]:8391")
```

2. [URL](https://nodejs.org/api/url.html) オブジェクトで `username` と `password` オプションを設定します。

```js
const proxyOpts = new URL("http://156.127.0.192:8391")

proxyOpts.username = "admin"

proxyOpts.password = "lK4w90MEe45YIkOpk"

const proxyAgent = new SocksProxyAgent(proxyOpts)
```

同じ方法はHttpsProxyAgentでも機能します。

### 環境変数でプロキシを設定する

Axiosでプロキシをグローバルに設定する別の方法として、次の環境変数を設定する方法があります。

- `HTTP_PROXY`：HTTPリクエストに使用するプロキシサーバーのURLです。
- `HTTPS_PROXY`：HTTPSリクエストに使用するプロキシサーバーのURLです。

LinuxまたはmacOSでは、次のように設定できます。

```bash
export HTTP_PROXY = "[<PROTOCOL>://]<USERNAME>:<PASSWORD>@<HOST>[:<PORT>]"

export HTTPS_PROXY = "[<PROTOCOL>://]<USERNAME>:<PASSWORD>@<HOST>[:<PORT>]"
```

Axiosがこれらの環境変数を検出すると、認証用の認証情報を含むプロキシ設定をそれらから読み取ります。Axiosにそれらの環境変数を無視させるには、`proxy` フィールドを `false` に設定してください。なお、プロキシを経由させないべきドメインのカンマ区切りリストとして `NO_PROXY` env を定義することも可能です。

同じ仕組みは、[cURLでプロキシを使用する](https://brightdata.jp/blog/proxy-101/curl-with-proxies) 場合にも機能します。

### ローテーティングプロキシを実装する

ターゲットサイトにプロキシのIPアドレスをブロックされないようにするには、実行する各リクエストが異なるプロキシサーバーから発信されるようにしてください。

1. それぞれが異なるプロキシへの接続情報を含むオブジェクトのリストを定義します。
2. 各リクエストの前にプロキシオブジェクトをランダムに選択します。
3. 選択したプロキシをAxiosで設定します。

上記のアプローチは、Bright Dataが提供する [ローテーティングプロキシ](https://brightdata.jp/solutions/rotating-proxies) のような、信頼性の高いプロキシサーバーのプールにアクセスできることを前提としています。

## 結論

Bright Dataは世界最高のプロキシサーバーを管理しており、Fortune 500企業や20,000社以上のお客様に提供しています。世界規模のプロキシネットワークには以下が含まれます。

*   [Datacenter proxies](https://brightdata.jp/proxy-types/datacenter-proxies) – 770,000超のデータセンターIP。
*   [Residential proxies](https://brightdata.jp/proxy-types/residential-proxies) – 195か国以上にわたる7,200万超のレジデンシャルIP。
*   [ISP proxies](https://brightdata.jp/proxy-types/isp-proxies) – 700,000超のISP IP。
*   [Mobile proxies](https://brightdata.jp/proxy-types/mobile-proxies) – 700万超のモバイルIP。

当社のプロキシサーバーを試すには、今すぐ[無料のBright Dataアカウントを作成](https://brightdata.jp/#popup-155639)してください。