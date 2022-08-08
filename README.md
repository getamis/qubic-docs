# Qubic Developer Documents


## APIs

* **Qubic Creator**
  * [Admin API (GraphQL)](./creator/README.md#qubic-creator-admin-api)
* **Qubic Pass**
  * [Admin API (GraphQL)](./pass/README.md#qubic-pass-admin-api)
  * [Client-side API (REST)](./pass/README.md#qubic-pass-client-api)


## Usages

Qubic Pass Client-side API 無需權限，只需透過按鈕或 Javascript 程式碼打開彈出視窗，讓使用者連結錢包。

Qubic Creator 以及 Qubic Pass 的 Admin API 是 server side 使用的 API，需要透過 API key 與 secret 獲取存取 API 的權限。

請參考下方文件了解如何產生 headers 以及使用範例。


### Generate API Headers <a id="headers" />

在使用之前，必須先申請取得 API key 和 secret，使用指定方法簽名後，放入 request 的 headers 中。

```ts
import HmacSHA256 from 'crypto-js/hmac-sha256';
import Base64 from 'crypto-js/enc-base64';

export function serviceHeaderBuilder(options) {
  const { url, apiKey, apiSecret, body = '' } = options;

  if (!apiKey || !apiSecret) {
    throw Error('apiKey and apiSecret should have value')
  }

  const now = Date.now();
  const msg = `${now}POST${url}${body}`;
  const sig = HmacSHA256(msg, apiSecret).toString(Base64);

  return {
    // CORS
    'sec-fetch-dest': 'empty',
    'sec-fetch-mode': 'cors',
    'sec-fetch-site': 'cross-site',
    // API Key
    'X-Qubic-Api-Key': apiKey,
    'X-Qubic-Ts': now.toString(),
    'X-Qubic-Sign': sig,
  };
}
```


### GraphQL Example

GraphQL 可以直接使用簡單的 fetch 來獲得資料，以下是 GraphQL 基金會提供的 API 查詢方式。

https://graphql.org/graphql-js/graphql-clients/

另外以下是兩種常用的 GraphQL 客戶端，以及使用範例。


#### Apollo Link

https://www.apollographql.com/docs/react/api/link/introduction/

```ts
const getApiAuthLink = (apiKey, apiSecret) =>
  setContext(async (request, previousContext) => {
    const { operationName, variables, query } = request;
    const { headers = {} } = previousContext;

    const body =
      operationName && query
        ? JSON.stringify({
            operationName,
            variables,
            query: print(query),
          })
        : '';

    const serviceHeaders = geServiceHeaders({
      url: GRAPHQL_URL,
      apiKey,
      apiSecret,
      body,
    });

    return {
      headers: {
        ...serviceHeaders,
        ...headers,
      },
    };
  });
```


#### GraphQL Request

https://github.com/prisma-labs/graphql-request

```ts
import request from 'graphql-request';

function resolveRequestDocument(document) {
  if (typeof document === 'string') {
    let operationName;

    try {
      const parsedDocument = parse(document);
      operationName = extractOperationName(parsedDocument);
    } catch (err) {
      // Failed parsing the document, the operationName will be undefined
    }
    return { query: document, operationName };
  }

  const operationName = extractOperationName(document);
  return { query: print(document), operationName };
}

export function requestGraphql({
  query,
  variables,
  apiKey,
  apiSecret,
  creatorUrl,
  isPublic = false,
}) {
  const { operationName, query: graphQLQuery } = resolveRequestDocument(graphQLQuery);
  const body =
    operationName && query
      ? JSON.stringify({
          query: graphQLQuery,
          variables,
          operationName,
        })
      : '';

  const headers = serviceHeaderBuilder({
    url: GRAPHQL_URL,
    apiKey,
    apiSecret,
    body,
  });

  return request({
    url: GRAPHQL_URL,
    document: query,
    variables,
    requestHeaders: headers,
  });
}
```
