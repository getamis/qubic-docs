# Creator Http Request

## Abstract

在使用之前，必須先申請取得 apiKey 和 apiSecret，
並在發送 request 在 headers 放入必要欄位

## 產生 Headers

```ts
import HmacSHA256 from 'crypto-js/hmac-sha256';
import Base64 from 'crypto-js/enc-base64';

export function serviceHeaderBuilder(options: {
  // POST, GET, PUT, DELETE, PATCH ...
  httpMethod: string;
  // endpoint 的完整 url
  serviceUri: string;
  // 申請而來的 apiKey, apiSecret 
  apiKey: string;
  apiSecret: string;
  // 傳送內文
  body?: string;
  // 有些 endpoint 需要 access token，可由 auth api 來取得
  accessToken?: string;
}): HeadersInit {
  const { httpMethod, serviceUri, body, apiKey, apiSecret, accessToken } = options;

  if (!apiKey || !apiSecret) {
    // apiKey 和 apiSecret 不可為空字串 `''`
    throw Error('apiKey and apiSecret should have value')
  }

  // 取得當前用戶時間
  const now = Date.now();

  // serviceUri ex: https://admin.qubic.market/market/services/graphql-public
  const urlObj = new URL(serviceUri);
  // `/market/services/graphql-public`
  const requestURI = `${urlObj.pathname}${urlObj.search}`;

  // 產生訊息
  const msg = `${now}${httpMethod}${requestURI}${body}`;
  // 用 apiSecret 對訊息簽名，並轉成 base64
  const sig = HmacSHA256(msg, apiSecret).toString(Base64);

  return {
    // CORS
    'sec-fetch-dest': 'empty',
    'sec-fetch-mode': 'cors',
    'sec-fetch-site': 'cross-site',

    // API Key
    'X-Es-Api-Key': apiKey,

    ...(body && {
      'X-Es-Encrypted': 'yes',
      'X-Es-Ts': now.toString(),
      'X-Es-Sign': sig,
    }),

    ...(accessToken && {
      'Access-Control-Allow-Credentials': 'true',
      Authorization: `Bearer ${accessToken}`,
    }),
  };
}
```

## Graphql Example

以下是用 graphql 時的範例

### Apollo link

[Apollo link](https://www.apollographql.com/docs/react/api/link/introduction/)

```ts
const getApiAuthLink = (apiKey: string, apiSecret: string) =>
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
        : undefined;

    const serviceHeaders = geServiceHeaders({
      httpMethod: HTTP_METHOD,
      serviceUri: BACKEND_GQL_URL,
      apiKey,
      apiSecret,
      body,
      accessToken: getAccessToken(),
    });

    return {
      headers: {
        ...serviceHeaders,
        ...headers,
      },
    };
  });
```

### Graphql Request

[Graphql Request](https://github.com/prisma-labs/graphql-request)

```ts
import request, { RequestDocument } from 'graphql-request';

function resolveRequestDocument(document: RequestDocument): { query: string; operationName?: string } {
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
}: // eslint-disable-next-line @typescript-eslint/no-explicit-any
RequestGraphqlInput): Promise<any> {
  const endPoint = `${creatorUrl}/services/graphql-${isPublic ? 'public' : 'acc'}`;

  const { operationName, query: graphQLQuery } = resolveRequestDocument(graphQLQuery);
  const body =
    operationName && query
      ? JSON.stringify({
          query: graphQLQuery,
          variables,
          operationName,
        })
      : undefined;

  const headers = serviceHeaderBuilder({
    serviceUri: endPoint,
    httpMethod: 'POST',
    apiKey,
    apiSecret,
    body,
  });

  return request({
    url: endPoint,
    document: query,
    variables,
    requestHeaders: headers,
  });
}
```
