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

```typescript
import HmacSHA256 from 'crypto-js/hmac-sha256';
import Base64 from 'crypto-js/enc-base64';

const createHeader = (options: { url: string; body: string }) => {
  const { url, body } = options;

  const urlObj = new URL(url);
  const resource = `${urlObj.pathname}${urlObj.search}`;

  const now = Date.now();
  const msg = `${now}POST${resource}${body}`;
  const sig = HmacSHA256(msg, QUBIC_API_SECRET).toString(Base64);

  return {
    'Content-Type': 'application/json',
    Accept: 'application/json',
    // CORS
    'sec-fetch-dest': 'empty',
    'sec-fetch-mode': 'cors',
    'sec-fetch-site': 'cross-site',
    // API Key
    'X-Qubic-Api-Key': QUBIC_API_KEY,
    'X-Qubic-Ts': now.toString(),
    'X-Qubic-Sign': sig,
  };
};
```


### GraphQL Example

以下建議使用兩種常用的 GraphQL 庫來存取 Qubic GraphQL API。

#### GraphQL Request

https://github.com/prisma-labs/graphql-request

*Install*

```
npm install graphql graphql-request
```

*Example*

```typescript
import graphqlRequest, { resolveRequestDocument } from 'graphql-request';

const request = async <T>(document: string, variables?: { [key: string]: any }): Promise<T> => {
  const { operationName, query } = resolveRequestDocument(document);
  const body =
    operationName && query
      ? JSON.stringify({
          query,
          variables,
          operationName,
        })
      : '';

  const headers = createHeader({
    url: GRAPHQL_BACKEND_URL,
    body,
  });

  return graphqlRequest<T>({
    url: GRAPHQL_BACKEND_URL,
    document,
    variables,
    requestHeaders: headers,
  });
};
```



#### Apollo Link

https://www.apollographql.com/docs/react/api/link/introduction/

```typescript
const getApiAuthLink = () =>
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

    const serviceHeaders = createHeader({
      url: GRAPHQL_URL,
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


