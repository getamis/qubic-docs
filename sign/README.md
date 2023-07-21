# Sign

## Sign Example

```sh
yarn add crypto-js      # yarn
npm install crypto-js   # npm
```

```js
import HmacSHA256 from "crypto-js/hmac-sha256.js";
import Base64 from "crypto-js/enc-base64.js";

function sign(option) {
  const { now, method, resource, secret } = option;

  const msg = `${now}${method}${resource}`;
  const sign = HmacSHA256(msg, secret).toString(Base64);

  return { msg, sign };
}

const { msg, sign } = sign({
  now: 1689907490132,
  method: "POST",
  resource: "/admin/graphql",
  secret: "secret",
});

console.log(msg === "1689907490132POST/admin/graphql"); // true
console.log(sign === "d1tZksk8khiWQ+UTUY7m6u1Msb5Oyhfej+c384e5GM8="); // true
```

## Secure Sign Example

```sh
yarn add crypto-js      # yarn
npm install crypto-js   # npm
```

```js
import HmacSHA256 from "crypto-js/hmac-sha256.js";
import Base64 from "crypto-js/enc-base64.js";

function signSec(option) {
  const { now, body, method, resource, secret } = option;

  const msg = `${now}${method}${resource}${body}`;
  const sign = HmacSHA256(msg, secret).toString(Base64);

  return { msg, sign };
}

const { msg, sign } = signSec({
  now: 1566549227549,
  body: "the_body",
  method: "PUT",
  resource: "/test/path?currency=USD",
  secret: "secret",
});

console.log(msg === "1566549227549PUT/test/path?currency=USDthe_body"); // true
console.log(sign === "xN/7FHzMvIVbJYESYPJlMwNHL9r3DBZ21lsjSn5W3Bo="); // true
```
