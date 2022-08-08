# Qubic Pass GraphQL API

## Admin API <a id="qubic-pass-admin-api" />

* Endpoint
  * https://pass.qubic.app/admin/graphql
* Try it in the playground
  * https://pass.qubic.app/admin/graphql/playground

## Client-side APIs (TBD) <a id="qubic-pass-client-api" />

### Sign In

```
https://pass.qubic.app/signIn?{query}
```

Get the signature from user by opening an auth window that allows users to connect wallet, including Qubic, Metamask or Wallet Connect.

Keep the signature and verify it on the server side to determine that the request was issued by the wallet owner, similar to the use of access token.

### Verify

```
https://pass.qubic.app/verify?{query}
```

Verify NFT token ownership by opening an auth window that allows users to connect wallet, including Qubic, Metamask or Wallet Connect.](https://pass.qubic.app/admin/graphql/playg)round