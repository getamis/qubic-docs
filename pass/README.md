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

透過開啟彈出的 Qubic Pass 視窗，使用者可以連結 Qubic、Metamask 或是 Wallet Connect 的錢包，並取回簽名與錢包地址等資訊。

將 `payload` 與 `signature` 妥善保管在 Cookies 或 Local Storage 中，便可以隨時驗證發出請求者是否為真實錢包擁有者。

如果要在伺服器端呼叫 Qubic Pass Admin API，也建議先透過 Client 端傳來的 `payload` 與 `signature` 再次驗證用戶。

#### query format

| Key        | Value           |                                                                                                                                                                         Remark |
| :--------- | :-------------- | -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| requestId  | `String(32)`    | 必須是 [uuidv4](https://zh.wikipedia.org/zh-tw/%E9%80%9A%E7%94%A8%E5%94%AF%E4%B8%80%E8%AF%86%E5%88%AB%E7%A0%81#%E7%89%88%E6%9C%AC4%EF%BC%88%E9%9A%8F%E6%9C%BA%EF%BC%89) 的格式 |
| customData | `String(< 500)` |                                                                                                                                             (optional)任意資料，給客戶端做使用 |

*開啟彈出視窗*

```typescript
import { v4 as uuidv4 } from 'uuid';
import qs from 'query-string';

const newRequestId = uuidv4();

const link = qs.stringifyUrl({
  url: `${QUBIC_PASS_BASE_URL}/signIn`,
  query: {
    requestId: newRequestId,
    customData: JSON.stringify({}),
  },
});

const popup = window.open(link, TARGET, 'location=no,resizable=yes,scrollbars=yes,status=yes,height=680,width=350');
const timer = setInterval(() => {
  if (popup && popup.closed) {
    clearInterval(timer);
  }
});
```

#### event data format

| Key       | Value                      |                Remark |
| :-------- | :------------------------- | --------------------: |
| payload   | `String`               |          本次 Request 內容 |
| signature | `String`            |   上述 payload 的簽名 |
| target    | `String` | 用以分辨 message 來源 |

*監聽簽名回傳資訊*

```typescript
const listener = (event: MessageEvent<SignInResponse>) => {
 const { data } = event;

 if (data.target !== TARGET || data.payload.requestId !== requestId) return;
 const newPlain = JSON.stringify(data.payload);
 const newSignature = data.signature;
 const newUserAddress = data.payload.userAddress;

 if (requestId === data.payload.requestId) {
   // do something
 }
};

window.addEventListener('message', listener);
return () => {
 window.removeEventListener('message', listener);
};
```

#### 如何驗證簽名

簽名是使用 ECDSA 橢圓曲線演算法產生，透過 ECRecovery 機制可以驗證簽名，以下使用 node.js 的 `ethereumjs-util` 庫作為範例。

註：如果曾經將 `payload` 回傳的資料保存在 Cookies 中，需要先透過 decodeURIComponent 還原原始的 `payload` 資料。

```typescript
import { fromRpcSig, ecrecover, bufferToHex, hashPersonalMessage, pubToAddress } from 'ethereumjs-util';
import { Buffer } from 'buffer';

export const verify = (payload: string, signature: string): string => {
  const { userAddress } = JSON.parse(payload);

  if (!userAddress) throw new Error('Invalid signature. Missing user address.');

  const messageHash = hashPersonalMessage(Buffer.from(payload));

  const { v, r, s } = fromRpcSig(signature);
  const publicKey = ecrecover(messageHash, v, r, s);

  const address = bufferToHex(pubToAddress(publicKey));

  if (address === String(userAddress).toLowerCase()) return userAddress;
  throw new Error('Invalid signature. Mismatching user address');
};
```

### Verify

```
https://pass.qubic.app/verify?{query}
```

透過開啟彈出的 Qubic Pass 視窗，使用者可以連結 Qubic、Metamask 或是 Wallet Connect 的錢包，並會觸發 Webhook 通知自訂的伺服器驗證結果。

因為支援 Webhook，此方法更適合沒有客製化前端的服務，例如：Line、Discord Chatbot，或是 Widget 等小工具。

此方法同時也支援帶入 `contractAddress` 與 `tokenId` 直接驗證錢包用戶是否為 NFT 持有者。
