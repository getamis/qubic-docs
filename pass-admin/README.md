# Pass Admin API

- GraphQL Endpoint
  - https://pass.qubic.app/admin/graphql
- GraphQL Playground
  - https://pass.qubic.app/admin/graphql/playground

## Introduction

這個範例將展示如何呼叫 Pass Admin API 中的 `user` query，你也可以前往 [GraphQL Playground](https://pass.qubic.app/admin/graphql/playground) 探索其他的 API 資源

## Usage

```sh
yarn add graphql-request graphql crypto-js      # yarn
npm install graphql-request graphql crypto-js   # npm
```

```js
import HmacSHA256 from "crypto-js/hmac-sha256.js";
import Base64 from "crypto-js/enc-base64.js";
import { request, gql } from "graphql-request";

const KEY = "YOUR_PASS_ADMIN_API_KEY";
const SECRET = "YOUR_PASS_ADMIN_API_SECRET";

const url = "https://pass.qubic.app/admin/graphql";

const urlObj = new URL(url);
const resource = `${urlObj.pathname}${urlObj.search}`; // output: /admin/graphql

const now = Date.now();
const msg = `${now}POST${resource}`;
const sig = HmacSHA256(msg, SECRET).toString(Base64);

const requestHeaders = {
  "x-qubic-api-key": KEY,
  "x-qubic-ts": now.toString(),
  "X-qubic-sign": sig,
};

const document = gql`
  query user($userAddress: Address!) {
    user(userAddress: $userAddress) {
      id
    }
  }
`;

const variables = {
  userAddress: "0x09F9aB1ae48CcBD39D4b07643b0CdEB658d97da5",
};

const result = await request({
  url,
  document,
  variables,
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

const KEY = "YOUR_PASS_ADMIN_API_KEY";
const SECRET = "YOUR_PASS_ADMIN_API_SECRET";

const url = "https://pass.qubic.app/admin/graphql";

const urlObj = new URL(url);
const resource = `${urlObj.pathname}${urlObj.search}`; // output: /admin/graphql

const document = gql`
  query user($userAddress: Address!) {
    user(userAddress: $userAddress) {
      id
    }
  }
`;

const variables = {
  userAddress: "0x09F9aB1ae48CcBD39D4b07643b0CdEB658d97da5",
};

const { operationName, query } = resolveRequestDocument(document);

const body = JSON.stringify({
  query,
  variables,
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
  variables,
  requestHeaders,
});

console.log(result);
```
