# Bot Integration Guide

## Contract Addresses (Monad Testnet)

```
BondingCurve: [DEPLOYED_ADDRESS]
BondingCurveRouter: [DEPLOYED_ADDRESS]
DexRouter: [DEPLOYED_ADDRESS]
Factory: 0x961235a9020B05C44DF1026D956D1F4D78014276
WMON: 0x760AfE86e5de5fa0Ee542fc7B7B713e1c5425701
```

## 1️⃣ Token Contract

- Standard ERC20 with permit
- Check `isListed()` to determine trading venue

## 2️⃣ BondingCurve (Direct)

```solidity
// Read states
curves(token) → (realMonReserve, realTokenReserve, virtualMonReserve, virtualTokenReserve, k, targetTokenAmount, initVirtualMonReserve, initVirtualTokenReserve)
isLocked(token) → bool
isListed(token) → bool


buy(to, token, amountOut) // Requires exact MON input
sell(to, token, amountOut) // Requires exact token input
```

## 3️⃣ BondingCurveRouter (Recommended for Bonding)

```solidity
// Price queries
getAmountOut(token, amountIn, is_buy) → amountOut
getAmountIn(token, amountOut, is_buy) → amountIn

// Buy tokens
buy(BuyParams{token, amountIn, amountOutMin, to, deadline})
exactOutBuy(ExactOutBuyParams{token, amountOut, amountInMax, to, deadline})

// Sell tokens
sell(SellParams{token, amountIn, amountOutMin, to, deadline})
exactOutSell(ExactOutSellParams{token, amountOut, amountInMax, to, deadline})

// With permit (save gas)
sellPermit(SellPermitParams{...})
exactOutSellPermit(ExactOutSellPermitParams{...})
```

## 4️⃣ DexRouter (After Listing)

```solidity
// Price queries (uses QuoterV2)
getAmountOut(token, amountIn, is_buy) → amountOut
getAmountIn(token, amountOut, is_buy) → amountIn
availableBuyTokens(token) → (availableBuyToken, requiredMonAmount)

// Trading
buy(BuyParams{token, amountOutMin, to, deadline}) payable
sell(SellParams{token, amountIn, amountOutMin, to, deadline})
exactOutBuy(ExactOutBuyParams{...})
exactOutSell(ExactOutSellParams{...})
```

### 1. Check Token Status

```javascript
const isListed = await bondingCurve.isListed(tokenAddress);
const isLocked = await bondingCurve.isLocked(tokenAddress);

if (!isListed && !isLocked) {
  // Trade on BondingCurveRouter
} else if (isListed) {
  // Trade on DexRouter
}
```

### 2. Get Prices

```javascript
// BondingCurveRouter
const amountOut = await bondingCurveRouter.getAmountOut(token, amountIn, true); // buy
const amountIn = await bondingCurveRouter.getAmountIn(token, amountOut, false); // sell
const [availableTokens, requiredMon] =
  await bondingCurveRouter.availableBuyTokens(token);

// DexRouter (returns via revert data)
try {
  await dexRouter.getAmountOut.staticCall(token, amountIn, true);
} catch (error) {
  const amountOut = ethers.AbiCoder.defaultAbiCoder().decode(
    ["uint256"],
    error.data
  )[0];
}
```

### 3. Execute Trade

```javascript
// Buy on BondingCurveRouter
await bondingCurveRouter.buy({
  token: tokenAddress,
  amountIn: ethers.parseEther("1"),
  amountOutMin: minTokens,
  to: myAddress,
  deadline: Math.floor(Date.now() / 1000) + 300,
});

// Sell on DexRouter
await dexRouter.sell({
  token: tokenAddress,
  amountIn: tokenAmount,
  amountOutMin: minMON,
  to: myAddress,
  deadline: Math.floor(Date.now() / 1000) + 300,
});
```
