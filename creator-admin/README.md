# Creator Admin API

- GraphQL Endpoint
  - https://creator.qubic.app/admin/graphql
- GraphQL Playground
  - https://creator.qubic.app/admin/graphql/playground

## Introduction

這個範例將展示如何呼叫 Creator Admin API 中的 `shop` query，你也可以前往 [GraphQL Playground](https://creator.qubic.app/admin/graphql/playground) 探索其他的 API 資源

## Usage

```sh
yarn add graphql-request graphql crypto-js      # yarn
npm install graphql-request graphql crypto-js   # npm
```

```js
import HmacSHA256 from "crypto-js/hmac-sha256.js";
import Base64 from "crypto-js/enc-base64.js";
import { request, gql } from "graphql-request";

const KEY = "YOUR_CREATOR_ADMIN_API_KEY";
const SECRET = "YOUR_CREATOR_ADMIN_API_SECRET";

const url = "https://creator.qubic.app/admin/graphql";

const urlObj = new URL(url);
const resource = `${urlObj.pathname}${urlObj.search}`; // output: /admin/graphql

const now = Date.now();
const msg = `${now}POST${resource}`;
const sig = HmacSHA256(msg, SECRET).toString(Base64);

const requestHeaders = {
  "x-qubic-api-key": KEY,
  "x-qubic-ts": now.toString(),
  "x-qubic-sign": sig,
};

const document = gql`
  query shop {
    shop {
      id
    }
  }
`;

const result = await request({
  url,
  document,
  requestHeaders,
});

console.log(result);
```

## Secure Usage

```sh
yarn add graphql-request graphql crypto-js      # yarn
npm install graphql-request graphql crypto-js   # npm
```

```js
import HmacSHA256 from "crypto-js/hmac-sha256.js";
import Base64 from "crypto-js/enc-base64.js";
import { request, gql, resolveRequestDocument } from "graphql-request";

const KEY = "YOUR_CREATOR_ADMIN_API_KEY";
const SECRET = "YOUR_CREATOR_ADMIN_API_SECRET";

const url = "https://creator.qubic.app/admin/graphql";

const urlObj = new URL(url);
const resource = `${urlObj.pathname}${urlObj.search}`; // output: /admin/graphql

const document = gql`
  query shop {
    shop {
      id
    }
  }
`;

const { operationName, query } = resolveRequestDocument(document);

const body = JSON.stringify({
  query,
  operationName,
});

const now = Date.now();
const msg = `${now}POST${resource}${body}`;
const sig = HmacSHA256(msg, SECRET).toString(Base64);

const requestHeaders = {
  "x-qubic-api-key": KEY,
  "x-qubic-ts": now.toString(),
  "x-qubic-sign": sig,
};

const result = await request({
  url,
  document,
  requestHeaders,
});

console.log(result);
```
