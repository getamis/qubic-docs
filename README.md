# Qubic Developer Documents

## API & SDK

- **Server-side**
  - [Creator Admin API (GraphQL)](./creator-admin/README.md)
  - [Pass Admin API (GraphQL)](./pass-admin/README.md)
- **Client-side**
  - [Creator Storefront API (GraphQL)](./creator-storefront/README.md)
  - Qubic Pass Client-side API：無需權限，只需透過按鈕或 Javascript 程式碼打開彈出視窗，讓使用者連結錢包
  - [Qubic Connect SDK](https://github.com/getamis/qubic-connect#readme)

## Introduction

Creator Admin API、Pass Admin API 以及 Creator Storefront API 要透過 key/secret 取得使用權限，需於通訊時在 header 中加入指定的屬性以完成驗證，描述如下：

| header 屬性     | 類型   | 說明               | 如何取得                                             |
| --------------- | ------ | ------------------ | ---------------------------------------------------- |
| x-qubic-api-key | string | API KEY            | [聯絡 Qubic 團隊](https://www.qubic.market/#contact) |
| x-qubic-ts      | string | 運算當下的時間戳記 | `Date.now().toString()`                              |
| x-qubic-sign    | string | 運算後的簽名       | [範例](#example)                                     |

## Example

基於 API 類型及安全性的要求，請參考下方文件的範例：

- Creator Admin API
  - [標準](./creator-admin/README.md#usage)
  - [安全性更高的 API 通訊機制](./creator-admin/README.md#secure-usage)
- Pass Admin API
  - [標準](./pass-admin/README.md#usage)
  - [安全性更高的 API 通訊機制](./pass-admin/README.md#secure-usage)
- Creator Storefront API
  - [標準](./creator-storefront/README.md#usage)
  - [安全性更高的 API 通訊機制](./creator-storefront/README.md#secure-usage)

## Troubleshooting

- 為何我一直收到 `{"error":{"code":404,"message":"resource not found"}}`？

  1.  請確認有將 [header 所需的資訊](#introduction)放入其中
  2.  請確認正在使用的 API key/secret 與 GraphQL endpoint 是匹配的
  3.  可以先試著寫[單機版程式驗證簽名](./sign/README.md)是否正確

## About GraphQL

GraphQL 是一種用於查詢和操作 API 的查詢語言，你可以直接使用 fetch(or axios) 向 Qubic GraphQL Server 發送請求

我們建議使用 [graphql-request](https://github.com/prisma-labs/graphql-request) 這套輕量的函式庫作為開發的工具，我們提供的[範例](#example)也會使用它進行介紹

如果你有 GraphQL 的開發經驗，也可以使用 [Apollo](https://www.apollographql.com/) 進行溝通，我們建議將邏輯以 [Apollo Link](https://www.apollographql.com/docs/react/api/link/introduction/) 的形式封裝
