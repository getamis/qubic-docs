# Creator Storefront API

- GraphQL Endpoint
  - https://creator.qubic.app/market/graphql
- GraphQL Playground
  - https://creator.qubic.app/market/graphql/playground

## Introduction

這個範例將展示如何呼叫 Creator Storefront API 中的 `now` query，你也可以前往 [GraphQL Playground](https://creator.qubic.app/market/graphql/playground) 探索其他的 API 資源

## Usage

```sh
yarn add graphql-request graphql crypto-js      # yarn
npm install graphql-request graphql crypto-js   # npm
```

```js
import HmacSHA256 from "crypto-js/hmac-sha256.js";
import Base64 from "crypto-js/enc-base64.js";
import { request, gql } from "graphql-request";

const KEY = "YOUR_STOREFRONT_API_KEY";
const SECRET = "YOUR_STOREFRONT_API_SECRET";

const url = "https://creator.qubic.app/market/graphql";

const urlObj = new URL(url);
const resource = `${urlObj.pathname}${urlObj.search}`; // output: /market/graphql

const now = Date.now();
const msg = `${now}POST${resource}`;
const sig = HmacSHA256(msg, SECRET).toString(Base64);

const requestHeaders = {
  "x-qubic-api-key": KEY,
  "x-qubic-ts": now.toString(),
  "X-qubic-sign": sig,
};

const document = gql`
  query now {
    now
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

const KEY = "YOUR_STOREFRONT_API_KEY";
const SECRET = "YOUR_STOREFRONT_API_SECRET";

const url = "https://creator.qubic.app/market/graphql";

const urlObj = new URL(url);
const resource = `${urlObj.pathname}${urlObj.search}`; // output: /market/graphql

const document = gql`
  query now {
    now
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
  "X-qubic-sign": sig,
};

const result = await request({
  url,
  document,
  requestHeaders,
});

console.log(result);
```
