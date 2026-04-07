Perfect — below is a **precise change log document** with exact file names and what was modified.
You can commit this as:

```
docs/adi-adiTestnet-registration.md
```

---

# ADI / adiTestnet Registration — Core → SDK → PWA

This document records all file-level changes required to support:

```
did:iden3:adi:adiTestnet:<genesis>
```

ChainId: `99999`
Network Flag: `249`

---

# 1️⃣ adi-id-js-core Changes

Repository:

```
adi-id-js-core/
```

All changes were made inside the **core registry layer**.

---

## 📁 File: `src/constants.ts`

### ✅ 1. Added ADI blockchain

**Location:** inside `export const Blockchain`

```ts
export const Blockchain: { [k: string]: string } = {
  Ethereum: 'eth',
  Polygon: 'polygon',
  Privado: 'privado',
  Billions: 'billions',
  Linea: 'linea',
  Unknown: 'unknown',
  NoChain: '',
  ReadOnly: 'readonly',
  Base: 'base',
  Bnb: 'bnb',

  // ✅ ADI
  Adi: 'adi',
};
```

---

### ✅ 2. Added adiTestnet network

**Location:** inside `export const NetworkId`

```ts
export const NetworkId: { [k: string]: string } = {
  Main: 'main',
  Mumbai: 'mumbai',
  Amoy: 'amoy',
  Goerli: 'goerli',
  Sepolia: 'sepolia',
  Zkevm: 'zkevm',
  Cardona: 'cardona',
  Test: 'test',
  Unknown: 'unknown',
  NoNetwork: '',

  // ✅ ADI
  AdiTestnet: 'adiTestnet',
};
```

---

### ✅ 3. Added ChainId mapping

**Location:** inside `export const ChainIds`

```ts
export const ChainIds: { [key: string]: number } = {
  // ... existing mappings

  // ✅ ADI
  [`${Blockchain.Adi}:${NetworkId.AdiTestnet}`]: 99999,
};
```

---

### ✅ 4. Added DID network flag mapping

**Location:** inside `blockchainNetworkMap`

```ts
const blockchainNetworkMap = {
  // ... existing mappings

  // ✅ ADI (network flag = 249)
  [`${Blockchain.Adi}:${NetworkId.AdiTestnet}`]: 249,
};
```

This ensures:

```
buildDIDType("iden3", "adi", "adiTestnet")
```

returns correct DID type bytes.

---

## 🧠 Why This File Matters

`src/constants.ts` defines:

* Blockchain names
* Network names
* Chain IDs
* Network flag byte
* DIDMethodNetwork registry

If ADI is not defined here:

```
Error: blockchain adi and network adiTestnet is not defined in core lib
```

---

# 2️⃣ adi-id-js-sdk Changes

Repository:

```
adi-id-js-sdk/
```

### ✅ Dependency updated to use forked core

**File:**

```
adi-id-js-sdk/package.json
```

Dependency must point to the forked core (not upstream):

Example (workspace):

```json
"@iden3/js-iden3-core": "workspace:*"
```

or

```json
"@iden3/js-iden3-core": "file:../adi-id-js-core"
```

Then rebuild:

```bash
npm install
npm run build
```

---

## 📁 File: `src/utils/did-helper.ts`

No logic change required.

This file already calls:

```ts
buildDIDType(method, blockchain, networkId)
```

Once core supports ADI, this automatically works.

---

# 3️⃣ adi-id-pwa Changes

Repository:

```
adi-id-pwa/
```

---

## 📁 File: `src/lib/adiNetwork.ts`

Runtime safety registration added:

```ts
export const ADI = {
  didMethod: "iden3",
  blockchain: "adi",
  networkId: "adiTestnet",
  chainId: 99999,
  networkFlag: 249,
};
```

And registration logic:

* registerDidMethod
* registerBlockchain
* registerNetwork
* registerDidMethodNetwork
* registerChainId
* initDIDParams()

This guarantees browser bundle consistency even if tables are tree-shaken.

---

## 📁 File: `src/wallet/did.ts`

DID generation uses:

```ts
const info = {
  method: "iden3",
  blockchain: ADI.blockchain,
  networkId: ADI.networkId,
};
```

Final DID format:

```
did:iden3:adi:adiTestnet:<genesis>
```

---

## 📁 File: `src/pages/Home.tsx`

Initialization:

```ts
ensureAdiNetworkRegistered();
const out = await getOrCreateHolderDID();
```

StrictMode-safe initialization guard added to prevent double execution in dev.

---

# 4️⃣ Final Working Flow

```
adi-id-js-core
    ↓ defines ADI in constants.ts
adi-id-js-sdk
    ↓ imports updated core
adi-id-pwa
    ↓ runtime ensures registration
    ↓ builds DID
```

Result:

```
did:iden3:adi:adiTestnet:2kBvhbF...
```

No more:

* unsupported blockchain
* unsupported DID method
* missing network flag
* DID build crashes

---

# 5️⃣ Important Constants Summary

| Field       | Value      |
| ----------- | ---------- |
| method      | iden3      |
| blockchain  | adi        |
| networkId   | adiTestnet |
| chainId     | 99999      |
| networkFlag | 249        |

---

# 6️⃣ What Was NOT Modified

We did NOT change:

* buildDIDType logic
* DID encoding logic
* Genesis encoding logic
* Credential issuance flow
* JWZ pack/unpack
* IdentityWallet

Those are handled separately.

---

If you'd like, I can now:

* Generate a clean git-style patch diff
* Produce a networkFlag bitwise-compliant version (matching upstream format)
* Or create a visual architecture diagram for the ADI stack
