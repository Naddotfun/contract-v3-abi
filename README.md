# Integration Guide

## Contract Addresses

### Monad Mainnet

```
DEX_FACTORY: 0xB5FC84346E8dFf79cbD08C19d7eC27c740664a57
WMON: 0x3bd359C1119dA7Da1D913D1C4D2B7c461115433A
QUOTER_V3: 0x2e06b20Ee1f785bCe6862971fdd58f7bA878da9C
CREATOR_MANAGER: 0xDe2E1c7ffDe10281ec3e515f12868eA7bCb1bF56
TOKEN_REGISTRY: 0x54467b6300474cf485829eA5151848AEb20d6138
FOUNDATION_TREASURY: 0xb200D8c25e37695Da63e6aE4BFE29eA5cC0EF340
COMMUNITY_TREASURY: 0x6AbCb85307C59FB6813F2E3f97509bc1D1ec87F8
CREATOR_TREASURY: 0x432cCB0C0fCBfEDdd392d95Aa64ebe5D78bE606e
TOKEN_TREASURY: 0x98f74cd9CDEdDd58E6460E8f6B89FCF8B102F8d7
BONDING_CURVE_ROUTER: 0x84202FD46851B4a9Da9Ab3100Dd459Bf1F0a3dC0
BONDING_CURVE: 0x7193E46d4a812c8990F96A6E9c1f1ed338b2a6b7
LP_MANAGER: 0xB555Cf21Cd738b2282B13c8ec9fF9B25bCCdffeB
UNISWAP_ACTOR: 0x39314025E1f0E2D430b65fb7d2A4a2D4Fd740576
DEX_DEPLOYER: 0x19eC7E26a4d3106dDe05ddb3690220881940d68D
DEX_ROUTER: 0x2b1d8Bb0dED32dd1C4D4B04B4B3829ddA9aecC65
REWARD_POOL: 0xE26Ef0fA5939439706199da23d3f7De42087f735
LENS: 0x5c34Ed2A9675DC4C77378175870a6822CB925A8B
TOKEN_IMPLEMENT: 0xaF84C7e1737540e6E30731D2e547a006F40b0C1c
```

### Monad Testnet

```
DEX_FACTORY: 0x99f4Aa293dcEfFA11aB0c03C359db45d05c7C863
WMON: 0x760AfE86e5de5fa0Ee542fc7B7B713e1c5425701
QUOTER_V3: 0xb9841fdAc1F0fD01C79095C3c97105D01273A192
CREATOR_MANAGER: 0x338394E774777e635841c45A222F1De09a6AD360
TOKEN_REGISTRY: 0x7f03e0b07f60b21a3E08205154D34fA2B83c655a
FOUNDATION_TREASURY: 0x8eb9B3b873b2260e95E9Be40594153A5f7B98628
COMMUNITY_TREASURY: 0x8C786190186518A3a95d86CeE8e972ce1C7Eda8b
CREATOR_TREASURY: 0x4312Ec1606F2977AfC9Be36C84E58b7087618818
TOKEN_TREASURY: 0xdEAc7461fdBf799Ad52E11413F267801AD544a11
BONDING_CURVE_ROUTER: 0xF57F14335e9670ed2C0CeF4A59fB707Cf2eB3FAC
BONDING_CURVE: 0xaD720f94689edB929D9be7613223320a0b2f260F
LP_MANAGER: 0x5Ab462177fE88A668e36eB0d36560A83036e0054
UNISWAP_ACTOR: 0x599F6c2307C4b90809F59275613Ab285D1B6bD80
DEX_DEPLOYER: 0x22CaB8b60EEfE9e0b7010C0B4c9C2822B32868c6
DEX_ROUTER: 0x34469738bbD2948F43E0e4B588BC5646BCf4a6bB
REWARD_POOL: 0xEbC10F2749Bf600e9F0b3D6eC7d762f8c9298Ba6
LENS: 0x1b2b500a6f6C8a25Ca0436d8183Ba25C9415e28E
TOKEN_IMPLEMENT: 0xEDc927B974F9964bba538bbD508F658d11917b43
```

---

## ⚙️ Network Configuration

### Testnet Parameters

| Parameter             | Value         | Description                         |
| --------------------- | ------------- | ----------------------------------- |
| Deploy Fee            | 50 MON        | Fee to create a new token           |
| Graduate Fee          | 3,000 MON     | Fee to graduate to DEX              |
| Virtual MON Reserve   | 90,000 MON    | Initial virtual MON for pricing     |
| Virtual Token Reserve | 1,073,000,191 | Initial virtual token supply        |
| Target Token Amount   | 279,900,191   | Tokens needed to trigger graduation |
| Total Token Supply    | 1,000,000,000 | Total supply per token              |

### Mainnet Parameters

**Until Monad Public Launch (Current Settings):**

| Parameter             | Value         | Description                         |
| --------------------- | ------------- | ----------------------------------- |
| Deploy Fee            | 0.0005 MON    | Fee to create a new token           |
| Graduate Fee          | 0.03 MON      | Fee to graduate to DEX              |
| Virtual MON Reserve   | 0.9 MON       | Initial virtual MON for pricing     |
| Virtual Token Reserve | 1,073,000,191 | Initial virtual token supply        |
| Target Token Amount   | 279,900,191   | Tokens needed to trigger graduation |
| Total Token Supply    | 1,000,000,000 | Total supply per token              |

**After Monad Public Launch (Future Settings):**

| Parameter             | Value         | Description                         |
| --------------------- | ------------- | ----------------------------------- |
| Deploy Fee            | 50 MON        | Fee to create a new token           |
| Graduate Fee          | 3,000 MON     | Fee to graduate to DEX              |
| Virtual MON Reserve   | 90,000 MON    | Initial virtual MON for pricing     |
| Virtual Token Reserve | 1,073,000,191 | Initial virtual token supply        |
| Target Token Amount   | 279,900,191   | Tokens needed to trigger graduation |
| Total Token Supply    | 1,000,000,000 | Total supply per token              |

**Key Differences:**

- 🧪 **Testnet**: Lower fees (1 MON) for easier testing
- 🚀 **Mainnet (Pre-Public)**: Very low fees (0.0005 MON deploy, 0.03 MON graduate) for initial launch period
- 🚀 **Mainnet (Post-Public)**: Production fees (50 MON deploy, 3,000 MON graduate)

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

// Bonding curve progress (basis points: 10000 = 100%)
getProgress(token) → progress

// Initial buy calculation (before token deployment)
getInitialBuyAmountOut(amountIn) → amountOut
```

**Key Features:**

- ✅ Single interface for all queries
- ✅ Auto-detects bonding curve vs DEX
- ✅ Returns correct router address
- ✅ All fees included in calculations
- ✅ Token status (graduated/locked)
- ✅ Available supply info
- ✅ Bonding curve progress tracking
- ✅ Pre-deployment buy estimation

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

## 1. CurveCreate
```solidity
emit CurveCreate(
    params.creator,
    token,
    pool,
    params.name,
    params.symbol,
    params.tokenURI,
    _config.virtualMonReserve,
    _config.virtualTokenReserve,
    _config.targetTokenAmount
);
```

- **When emitted**: When a new token is created
- **Meaning**: A new token with a bonding curve has been created
- **Included information**: Creator address, token address, pool address, token info (name, symbol, URI), initial virtual reserves, target token amount

---

## 2. CurveBuy
```solidity
emit CurveBuy(to, token, actualAmountIn, effectiveAmountOut);
```

- **When emitted**: When a user buys tokens
- **Meaning**: A token purchase has occurred on the bonding curve
- **Included information**: Recipient address, token address, actual wMon amount deposited, actual token amount received

---

## 3. CurveSell
```solidity
emit CurveSell(to, token, actualAmountIn, effectiveAmountOut);
```

- **When emitted**: When a user sells tokens
- **Meaning**: A token sale has occurred on the bonding curve
- **Included information**: Recipient address, token address, token amount sold, actual Mon amount received (after fees)

---

## 4. CurveSync
```solidity
emit CurveSync(
    token,
    curve.realMonReserve,
    curve.realTokenReserve,
    curve.virtualMonReserve,
    curve.virtualTokenReserve
);
```

- **When emitted**: After each buy/sell when the bonding curve state is updated
- **Meaning**: Synchronizes the current reserve state of the bonding curve
- **Included information**: Token address, real Mon reserve, real token reserve, virtual Mon reserve, virtual token reserve

---

## 5. CurveTokenLocked
```solidity
emit CurveTokenLocked(token);
```

- **When emitted**: When the token's virtual reserve reaches the target amount (targetTokenAmount)
- **Meaning**: The bonding curve is locked and no more buys/sells are possible. Now entering the DEX listing preparation phase
- **Included information**: Locked token address

---

## 6. CurveGraduate
```solidity
emit CurveGraduate(token, pool);
```

- **When emitted**: When a locked token is officially listed on the DEX (Uniswap V3)
- **Meaning**: Graduated from the bonding curve and liquidity has been transferred to the DEX
- **Included information**: Token address, DEX pool address

---

## Overall Flow Summary

1. **Create** → Token creation
2. **Buy/Sell** → Trading on the bonding curve (Sync event emitted after each trade)
3. **Locked** → Bonding curve locked when target is reached
4. **Graduated** → Liquidity migration to DEX completed

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

// Get bonding curve progress
const progress = await lens.getProgress(tokenAddress);
console.log(`Progress: ${(progress / 100).toFixed(2)}%`); // 5000 → 50.00%
```

### 3. Initial Buy (Before Token Deployment)

```javascript
// Calculate expected tokens for initial buy (before token exists)
const expectedTokens = await lens.getInitialBuyAmountOut(
  ethers.parseEther("1") // 1 MON
);

console.log(`You'll receive: ${ethers.formatEther(expectedTokens)} tokens`);

// Use this when creating token with initial buy
const deployFee = ethers.parseEther("1"); // 1 MON on testnet
const initialBuyMon = ethers.parseEther("1");

const tx = await bondingCurveRouter.create(
  {
    name: "MyToken",
    symbol: "MTK",
    tokenURI: "https://example.com/metadata.json",
    amountOut: expectedTokens, // Use calculated amount
    salt: ethers.randomBytes(32),
    actionId: 0,
  },
  { value: deployFee + initialBuyMon }
);
```

### 4. Buy Tokens (Recommended Pattern)

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

### 5. Sell Tokens

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

### 6. Exact Output Buy (Want Exact Tokens)

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

### 7. Sell with Permit (Save Gas)

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
