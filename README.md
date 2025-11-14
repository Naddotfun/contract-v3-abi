# Integration Guide

## Contract Addresses (Monad Testnet)

```
Lens: 0x9bd553886E1efc8bb9899d17160d92758fCBcDb5
BondingCurveRouter: 0xA8213c4853ab9101Fa5B06b039ED434266bC2392
BondingCurve: 0xb9B24593DE375c08c498672001939249b7C5D265
DexRouter: 0xC849F9a0a906a3bfd837ef1c961f524C69c2376b
Factory: 0x961235a9020B05C44DF1026D956D1F4D78014276
WMON: 0x760AfE86e5de5fa0Ee542fc7B7B713e1c5425701
```

---

## 🔍 Lens (Price Queries & Token Info)

**Use Lens for ALL read operations - single interface for everything**

```solidity
// Price queries (automatically detects bonding curve or DEX)
getAmountOut(token, amountIn, isBuy) → (router, amountOut)
getAmountIn(token, amountOut, isBuy) → (router, amountIn)

// Token status
isGraduated(token) → isGraduated
isLocked(token) → isLocked

// Available tokens to buy (bonding curve only)
availableBuyTokens(token) → (availableBuyToken, requiredMonAmount)
```

**Key Features:**

- ✅ Single interface for all queries
- ✅ Auto-detects bonding curve vs DEX
- ✅ Returns correct router address
- ✅ All fees included in calculations
- ✅ Token status (graduated/locked)
- ✅ Available supply info

**Why use Lens?**

- All read operations in one place
- No need to call different contracts
- Cleaner, simpler code
- Future-proof (supports new features automatically)

---

## 🛒 BondingCurveRouter (Bonding Curve Trading)

**Focus: Trading before graduation**

```solidity
// Token creation
create(TokenCreationParams{name, symbol, tokenURI, amountOut, salt, actionId}) payable → (token, pool)

// Trading
buy(BuyParams{amountOutMin, token, to, deadline}) payable
exactOutBuy(ExactOutBuyParams{amountInMax, amountOut, token, to, deadline}) payable
sell(SellParams{amountIn, amountOutMin, token, to, deadline})
exactOutSell(ExactOutSellParams{amountInMax, amountOut, token, to, deadline})

// With permit (gas-saving)
sellPermit(SellPermitParams{amountIn, amountOutMin, amountAllowance, token, to, deadline, v, r, s})
exactOutSellPermit(ExactOutSellPermitParams{amountInMax, amountOut, amountAllowance, token, to, deadline, v, r, s})
```

---

## 🏦 DexRouter (DEX Trading After Graduation)

**Focus: Trading after graduation**

```solidity
// Trading
buy(BuyParams{amountOutMin, token, to, deadline}) payable → amountOut
exactOutBuy(ExactOutBuyParams{amountInMax, amountOut, token, to, deadline}) payable → amountIn
sell(SellParams{amountIn, amountOutMin, token, to, deadline}) → amountOut
exactOutSell(ExactOutSellParams{amountInMax, amountOut, token, to, deadline}) → amountIn

// With permit (gas-saving)
sellPermit(SellPermitParams{amountIn, amountOutMin, amountAllowance, token, to, deadline, v, r, s}) → amountOut
exactOutSellPermit(ExactOutSellPermitParams{amountInMax, amountOut, amountAllowance, token, to, deadline, v, r, s}) → amountIn
```

---

## 📡 Events

### BondingCurve Events

```solidity
event CurveCreate(
    address indexed creator,
    address indexed token,
    address indexed pool,
    string name,
    string symbol,
    string tokenURI,
    uint256 virtualMon,
    uint256 virtualToken,
    uint256 targetTokenAmount
);

event CurveBuy(
    address indexed sender,
    address indexed token,
    uint256 amountIn,
    uint256 amountOut
);

event CurveSell(
    address indexed sender,
    address indexed token,
    uint256 amountIn,
    uint256 amountOut
);

event CurveTokenLocked(
    address indexed token
);

event CurveGraduate(
    address indexed token,
    address indexed pool
);
```

---

## 📚 Integration Examples

### 1. Create Token

```javascript
const tx = await bondingCurveRouter.create(
  {
    name: "MyToken",
    symbol: "MTK",
    tokenURI: "https://example.com/metadata.json",
    amountOut: ethers.parseEther("1000"), // Initial buy
    salt: ethers.randomBytes(32),
    actionId: 0,
  },
  { value: ethers.parseEther("0.1") } // Deploy fee + initial buy
);

const receipt = await tx.wait();
// Parse CurveCreate event for token address
```

### 2. Get Price (Using Lens)

```javascript
// Lens handles everything - bonding curve or DEX
const [router, amountOut] = await lens.getAmountOut(
  tokenAddress,
  ethers.parseEther("1"), // 1 MON input
  true // isBuy
);

console.log(`Router to use: ${router}`);
console.log(`Expected tokens: ${ethers.formatEther(amountOut)}`);

// For selling
const [routerSell, monOut] = await lens.getAmountOut(
  tokenAddress,
  ethers.parseEther("1000"), // 1000 tokens input
  false // isSell
);

console.log(`Expected MON: ${ethers.formatEther(monOut)}`);

// Check token status
const isGrad = await lens.isGraduated(tokenAddress);
const isLock = await lens.isLocked(tokenAddress);
console.log(`Graduated: ${isGrad}, Locked: ${isLock}`);

// Check available tokens (bonding curve only)
if (!isGrad) {
  const [available, required] = await lens.availableBuyTokens(tokenAddress);
  console.log(`Available: ${ethers.formatEther(available)} tokens`);
  console.log(`Required: ${ethers.formatEther(required)} MON`);
}
```

### 3. Buy Tokens (Recommended Pattern)

```javascript
// Step 1: Get price from Lens
const [router, expectedOut] = await lens.getAmountOut(
  tokenAddress,
  ethers.parseEther("1"), // 1 MON
  true
);

// Step 2: Execute on correct router (Lens tells you which one)
const isBondingCurve = router === bondingCurveRouter.address;
const minOut = (expectedOut * 99n) / 100n; // 1% slippage

if (isBondingCurve) {
  await bondingCurveRouter.buy(
    {
      amountOutMin: minOut,
      token: tokenAddress,
      to: myAddress,
      deadline: Math.floor(Date.now() / 1000) + 300,
    },
    { value: ethers.parseEther("1") }
  );
} else {
  await dexRouter.buy(
    {
      amountOutMin: minOut,
      token: tokenAddress,
      to: myAddress,
      deadline: Math.floor(Date.now() / 1000) + 300,
    },
    { value: ethers.parseEther("1") }
  );
}
```

### 4. Sell Tokens

```javascript
// Step 1: Get price from Lens
const [router, expectedMon] = await lens.getAmountOut(
  tokenAddress,
  ethers.parseEther("1000"), // Selling 1000 tokens
  false // isSell
);

// Step 2: Approve tokens to the router Lens returned
await token.approve(router, ethers.parseEther("1000"));

// Step 3: Execute sell
const isBondingCurve = router === bondingCurveRouter.address;
const minMon = (expectedMon * 99n) / 100n; // 1% slippage

if (isBondingCurve) {
  await bondingCurveRouter.sell({
    amountIn: ethers.parseEther("1000"),
    amountOutMin: minMon,
    token: tokenAddress,
    to: myAddress,
    deadline: Math.floor(Date.now() / 1000) + 300,
  });
} else {
  await dexRouter.sell({
    amountIn: ethers.parseEther("1000"),
    amountOutMin: minMon,
    token: tokenAddress,
    to: myAddress,
    deadline: Math.floor(Date.now() / 1000) + 300,
  });
}
```

### 5. Exact Output Buy (Want Exact Tokens)

```javascript
// Want exactly 1000 tokens
const [router, requiredMon] = await lens.getAmountIn(
  tokenAddress,
  ethers.parseEther("1000"), // Exact tokens wanted
  true
);

const isBondingCurve = router === bondingCurveRouter.address;
const maxMon = (requiredMon * 101n) / 100n; // 1% slippage

if (isBondingCurve) {
  await bondingCurveRouter.exactOutBuy(
    {
      amountInMax: maxMon,
      amountOut: ethers.parseEther("1000"),
      token: tokenAddress,
      to: myAddress,
      deadline: Math.floor(Date.now() / 1000) + 300,
    },
    { value: maxMon }
  );
} else {
  await dexRouter.exactOutBuy(
    {
      amountInMax: maxMon,
      amountOut: ethers.parseEther("1000"),
      token: tokenAddress,
      to: myAddress,
      deadline: Math.floor(Date.now() / 1000) + 300,
    },
    { value: maxMon }
  );
}
```

### 6. Sell with Permit (Save Gas)

```javascript
// Get price from Lens first
const [router, expectedMon] = await lens.getAmountOut(
  tokenAddress,
  ethers.parseEther("1000"),
  false
);

// Get permit signature
const deadline = Math.floor(Date.now() / 1000) + 3600;
const { v, r, s } = await getPermitSignature(
  token,
  owner,
  router, // Use router from Lens
  ethers.parseEther("1000"),
  deadline
);

// Sell with permit (no approve tx needed!)
const isBondingCurve = router === bondingCurveRouter.address;

if (isBondingCurve) {
  await bondingCurveRouter.sellPermit({
    amountIn: ethers.parseEther("1000"),
    amountOutMin: (expectedMon * 99n) / 100n,
    amountAllowance: ethers.parseEther("1000"),
    token: tokenAddress,
    to: myAddress,
    deadline: deadline,
    v,
    r,
    s,
  });
} else {
  await dexRouter.sellPermit({
    amountIn: ethers.parseEther("1000"),
    amountOutMin: (expectedMon * 99n) / 100n,
    amountAllowance: ethers.parseEther("1000"),
    token: tokenAddress,
    to: myAddress,
    deadline: deadline,
    v,
    r,
    s,
  });
}
```

---

## 🏗️ Architecture

```
┌──────────────────────────┐
│    User / Frontend       │
└────────────┬─────────────┘
             │
             ↓
┌────────────────────────────┐
│   Lens (All Queries)       │ ← Use this for everything!
│  - getAmountOut()          │
│  - getAmountIn()           │
│  → Returns router + amount │
└────────────┬───────────────┘
             │
    ┌────────┴────────┐
    ↓                 ↓
┌─────────────┐  ┌─────────────┐
│BondingCurve │  │  DexRouter  │
│   Router    │  │             │
│ (Pre-Grad)  │  │ (Post-Grad) │
└─────────────┘  └─────────────┘
```

---

## ✅ Best Practices

### 1. Always Use Lens for Queries

```javascript
// ✅ Good - Use Lens
const [router, amount] = await lens.getAmountOut(token, input, isBuy);

// ❌ Bad - Don't call routers directly for prices
const amount = await bondingCurveRouter.getAmountOutWithFee(...); // Don't do this
```

### 2. Use Lens for Token Status

```javascript
// ✅ Good - Use Lens for all status checks
const isGrad = await lens.isGraduated(token);
const isLock = await lens.isLocked(token);
const [available, required] = await lens.availableBuyTokens(token);

// ❌ Bad - Don't call BondingCurve directly
const isGrad = await bondingCurve.isGraduated(token); // Don't do this
```

### 3. Let Lens Choose the Router

```javascript
// ✅ Good - Lens returns correct router
const [router, price] = await lens.getAmountOut(...);
if (router === bondingCurveRouter.address) { /* use bonding curve */ }

// ❌ Bad - Manual selection based on status
const isGrad = await lens.isGraduated(token);
if (isGrad) { /* manually choose dex */ } // Less optimal
```

### 4. Calculate Slippage Properly

```javascript
// ✅ Good - Use bigint math
const minOut = (expectedOut * 99n) / 100n; // 1% slippage

// ❌ Bad - Floating point (precision loss)
const minOut = expectedOut * 0.99; // Don't do this
```

### 5. Use Permit to Save Gas

```javascript
// ✅ Good - One transaction (permit + sell)
await router.sellPermit({...});

// ❌ Acceptable but costs more gas - Two transactions
await token.approve(router, amount);
await router.sell({...});
```

---

## 🚨 Error Handling

### Lens

- No errors (pure view function)

### BondingCurveRouter

- `DeadlineExpired` - Transaction took too long
- `InsufficientAmountOut` - Slippage too high
- `InsufficientAmountInMax` - Max input exceeded
- `InsufficientMon` - Not enough MON sent
- `InvalidAllowance` - Invalid permit signature

### DexRouter

- `ExpiredDeadLine` - Transaction took too long
- `InsufficientOutput` - Slippage too high
- `InsufficientInput` - Not enough input
- `InvalidAmountIn/Out` - Invalid amounts

---

## 💡 Summary

**Before Lens:**

```javascript
// Check graduation
const isGrad = await bondingCurve.isGraduated(token);

// Get price from correct router
let price;
if (isGrad) {
  price = await dexRouter.getAmountOut(...);
} else {
  price = await bondingCurveRouter.getAmountOutWithFee(...);
}

// Execute on correct router
if (isGrad) {
  await dexRouter.buy(...);
} else {
  await bondingCurveRouter.buy(...);
}
```

**With Lens (Now):**

```javascript
// Get price AND router in one call
const [router, price] = await lens.getAmountOut(token, input, true);

// Execute on router returned by Lens
if (router === bondingCurveRouter.address) {
  await bondingCurveRouter.buy(...);
} else {
  await dexRouter.buy(...);
}
```

**Even Better - Helper Function:**

```javascript
async function buy(token, monAmount) {
  const [router, expectedOut] = await lens.getAmountOut(token, monAmount, true);
  const minOut = (expectedOut * 99n) / 100n;

  const routerContract =
    router === bondingCurveRouter.address ? bondingCurveRouter : dexRouter;

  return routerContract.buy(
    {
      amountOutMin: minOut,
      token,
      to: myAddress,
      deadline: Math.floor(Date.now() / 1000) + 300,
    },
    { value: monAmount }
  );
}

// Usage - super simple!
await buy(tokenAddress, ethers.parseEther("1"));
```
