# SatsTerminal SDK

A TypeScript/JavaScript SDK for interacting with the SatsTerminal ecosystem. This SDK allows developers to integrate SatsTerminal functionalities into their Node.js applications easily.

## Version Changelog

### v1.6.7
- Released MagicEden Sell support
- Made required runeName non-optional in type definitions


### v1.6.6

- Added `fill` parameter to `fetchQuote` method to automatically use the best-priced AMM when no orders are found in specified marketplaces

### v1.6.4 & v1.6.5

- PSBT inputs to sign for non RBF protected orders added to PSBT response
- Improved type definitions

### v1.6.3

- Added RBF protection support for rune transactions

## Table of Contents

- [Installation](#installation)
- [Environment Variables](#environment-variables)
- [Quick Start](#quick-start)
- [API Reference](#api-reference)
  - [Configuration](#configuration)
  - [Methods](#methods)
    - [signIn](#signin)
    - [bind](#bind)
    - [points](#points)
    - [search](#search)
    - [popularCollections](#popularcollections)
    - [fetchQuote](#fetchquote)
    - [getPSBT](#getpsbt)
    - [confirmPSBT](#confirmpsbt)
- [Example Flow](#example-flow)
- [Error Handling](#error-handling)
- [Common Issues](#common-issues)

## Installation

Install the SDK via npm:

```bash
npm install satsterminal-sdk@latest
```

## Environment Variables

The SDK requires configuration including an API key (which you can get from the SatsTerminal Admin Dashboard). You can set this up using environment variables or pass it directly to the constructor.

### Setting Environment Variables

Create a `.env` file in your project root:

```env
SATSTERMINAL_API_KEY=your_api_key_here
```

Load it using dotenv:

```javascript
require('dotenv').config();
```

## Quick Start

```typescript
import { SatsTerminal } from 'satsterminal-sdk';

const satsTerminal = new SatsTerminal({
  apiKey: 'your_api_key_here'
});

// Example: Searching for runes
const searchResult = await satsTerminal.search({ rune_name: 'DOG•GO•TO•THE•MOON' });
```

## API Reference

### Configuration

The SDK requires configuration when initializing:

```typescript
interface SatsTerminalConfig {
  apiKey: string;
}
```

### Methods

#### signIn

Register and authenticate a user with their Bitcoin and Ordinals addresses.

```typescript
async signIn(params: SignInParams): Promise<SignInResponse>
```

**Parameters:**

- `ord_address` (string): Ordinals address
- `btc_address` (string): Bitcoin address
- `ord_public_key` (string): Ordinals public key
- `btc_public_key` (string): Bitcoin public key
- `provider` (string): Wallet provider (e.g., 'xverse', 'unisat')

**Example:**

```typescript
const signIn = await satsTerminal.signIn({
  ord_address: "bc1...",
  btc_address: "3Pc...",
  ord_public_key: "...",
  btc_public_key: "...",
  provider: "xverse"
});
```

#### bind

Bind a user's wallet to Unisat and DotSwap. This is required only once before performing transactions.

```typescript
async bind(params: BindParams): Promise<BindResponse>
```

**Parameters:**

- `btcAddress` (string): Bitcoin address
- `nftAddress` (string): Ordinals address
- `sign` (string): User's signature of the bind message

**Example:**

```typescript
const bind = await satsTerminal.bind({
  btcAddress: "3Pc...",
  nftAddress: "bc1...",
  sign: "user_signature_here"
});
```

#### points

Get the user's accumulated Amber points balance.

```typescript
async points(params: PointsParams): Promise<PointsResponse>
```

**Parameters:**

- `ord_address` (string): Ordinals address

**Example:**

```typescript
const points = await satsTerminal.points({
  ord_address: "bc1..."
});
```

#### search

Search for runes by name.

```typescript
async search(params: SearchParams): Promise<SearchResponse>
```

**Parameters:**

- `rune_name` (string): The name of the rune to search for
- `sell` (boolean, optional): Whether to search for sell orders

**Example:**

```typescript
const result = await satsTerminal.search({
  rune_name: 'LOBO•THE•WOLF•PUP',
  sell: false
});
```

#### popularCollections

Fetch popular rune collections.

```typescript
async popularCollections(params: PopularCollectionsParams): Promise<PopularCollectionsResponse>
```

**Example:**

```typescript
const collections = await satsTerminal.popularCollections();
```

#### fetchQuote

Fetch a quote for a rune purchase.

```typescript
async fetchQuote(params: QuoteParams): Promise<QuoteResponse>
```

**Parameters:**

- `btcAmount` (number/string): Amount of BTC to spend
- `runeName` (string): Name of the rune
- `address` (string): Bitcoin address
- `themeID?` (string): Optional widget theme ID
- `sell?` (boolean): Optional flag for sell orders (default: false)
- `marketplaces?` (string[]): Optional list of marketplaces to use (default: empty, uses all available)
- `rbfProtection?` (boolean): Optional RBF protection flag (default: false)
- `fill?` (boolean): Optional flag that, when true, will use the best AMM to fulfill the order if no order is found in the specified marketplaces (default: false)

**Example:**

```typescript
const quote = await satsTerminal.fetchQuote({
  btcAmount: 0.0001,
  runeName: "LOBO•THE•WOLF•PUP",
  address: "bc1...",
  marketplaces: ["MagicEden"],
  rbfProtection: true,
  fill: true
});
```

**Example Response:**

```javascript
{
  bestMarketplace: 'MagicEden',
  selectedOrders: [
    {
      amount: '2039350000000',
      formattedAmount: '20393.5',
      id: '5b35441a-297b-45b4-8277-a3048309837b',
      isPending: false,
      rune: 'LOBOTHEWOLFPUP',
      maker: 'bc1p97mujnqerk098gnd9actkpev6waeauvzk2yt3zznllshz5evz66q55vjw6',
      makerReceiveAddress: '393PuAEiJ7JTFmVvkJJSG42s7241gG1ncC',
      makerFeeBps: 0,
      price: 10000,
      side: 'sell',
      status: 'valid',
      formattedUnitPrice: '0.490352',
      utxoIds: [
        'f99df727b7268bd4662798a8893d965940a63a68b77bdcc92d924ccecee66035:0'
      ],
      filledAmount: '0',
      formattedFilledAmount: '0',
      rbfProtectionShield: true,
      rbfProtectedSacAddress: 'bc1pg5lj78n75nhmvv47xq8y0kf6wzzed6ewesuq9p4qu6hmsnketz9sgmfdeh',
      market: 'MagicEden'
    }
  ],
  totalFormattedAmount: '20393.5',
  totalPrice: '0.0001',
  metrics: { /* metrics data */ },
  marketplaces: { /* marketplace data */ }
}
```

#### getPSBT

Generate a Partially Signed Bitcoin Transaction (PSBT).

```typescript
async getPSBT(params: GetPSBTParams): Promise<PSBTResponse>
```

**Parameters:**

- `orders` (RuneOrder[]): Array of orders to include (from fetchQuote)
- `address` (string): Bitcoin address
- `publicKey` (string): Bitcoin public key
- `paymentAddress` (string): Payment address
- `paymentPublicKey` (string): Payment public key
- `runeName` (string): Rune name
- `utxos?` (UTXO[]): Optional UTXOs to use
- `feeRate?` (number): Optional fee rate
- `slippage?` (number): Optional slippage percentage
- `themeID?` (string): Optional widget theme ID
- `sell?` (boolean): Optional flag for sell orders (default: false)
- `rbfProtection?` (boolean): Optional RBF protection flag (default: false)

**Example:**

```typescript
const psbtResponse = await satsTerminal.getPSBT({
  orders: quote.selectedOrders,
  address: "bc1...",
  publicKey: "...",
  paymentAddress: "3Pc...",
  paymentPublicKey: "...",
  feeRate: 5,
  runeName: "LOBO•THE•WOLF•PUP",
  slippage: 9,
  rbfProtection: true
});
```

**Example Response:**

```javascript
{
  marketplace: 'magiceden',
  swapId: '0b0fef9zmab9-bocn3afkmr',
  validOrders: [ '5b35441a-297b-45b4-8277-a3048309837b' ],
  inputs: [ 0 ],
  rbfProtected: {
    base64: 'cHNidP8BAKkCAAAAASdY5eq28qE1btcTbldfKCNVaaBNEBo6EYpvVKrVBaGqAQAAAAD/////AyMqAAAAAAAAIlEg1pNbOdpsqSDkdCcK+uN7LUaajvtMQqFlLaugrMI2cvSSBwAAAAAAACJRINaTWznabKkg5HQnCvrjey1Gmo77TEKhZS2roKzCNnL0VbIAAAAAAAAXqRTwcmaf/igf0hSlNZd3ss2y7lyJpYcAAAAACPwCbWUDc2lnQHpbBxAVPvbM3I975/SZDbOrmdw+3KjUk0gxbYeYxwVqAHdIYFxtfXbHHW5lMHU51KNrFa+h0OFOfVHVpSHUL/gJ/AJtZQRwZmVlAzU4MAv8Am1lBnNpZ2V4cAhCeVg6nAVwABT8Am1lD3RyYW5zYWN0aW9uVHlwZQdyYmYtYnV5GPwCbWUTdGFrZXJQYXltZW50QWRkcmVzcyIzUGNQOUo2QzlaTlFwVFJIcnRxRkJoUDNMb0JqU21Na1pxGPwCbWUTdGFrZXJSZWNlaXZlQWRkcmVzcz5iYzFwdXhhMmN3dmZ3NDd2YWFkdDRsZnh3Z2w0emxuOWVwcXRmc3EzeTdjbDd2cjV3dmZheXo2czIybW13cAABASAS5QAAAAAAABepFPByZp/+KB/SFKU1l3eyzbLuXImlhwEEFgAUY9wnP/vXXCJ9d3tDwzvzOL1cNV8ACfwCbWUEbWZlZQQAAAAACfwCbWUEdGZlZQQAAABkEPwCbWULaXRlbUZlZVRpcHMFMTAwLDAQ/AJtZQtydW5lT3JkZXJJZCQ1YjM1NDQxYS0yOTdiLTQ1YjQtODI3Ny1hMzA0ODMwOTgzN2IAAAA=',
    hex: '70736274ff0100a902000000012758e5eab6f2a1356ed7136e575f28235569a04d101a3a118a6f54aad505a1aa0100000000ffffffff03232a000000000000225120d6935b39da6ca920e474270afae37b2d469a8efb4c42a1652daba0acc23672f49207000000000000225120d6935b39da6ca920e474270afae37b2d469a8efb4c42a1652daba0acc23672f455b200000000000017a914f072669ffe281fd214a5359777b2cdb2ee5c89a5870000000008fc026d6503736967407a5b0710153ef6ccdc8f7be7f4990db3ab99dc3edca8d49348316d8798c7056a007748605c6d7d76c71d6e65307539d4a36b15afa1d0e14e7d51d5a521d42ff809fc026d650470666565033538300bfc026d6506736967657870084279583a9c05700014fc026d650f7472616e73616374696f6e54797065077262662d62757918fc026d651374616b65725061796d656e74416464726573732233506350394a3643395a4e517054524872747146426850334c6f426a536d4d6b5a7118fc026d651374616b657252656365697665416464726573733e6263317075786132637776667734377661616474346c667877676c347a6c6e3965707174667371337937636c3776723577766661797a367332326d6d77700001012012e500000000000017a914f072669ffe281fd214a5359777b2cdb2ee5c89a587010416001463dc273ffbd75c227d777b43c33bf338bd5c355f0009fc026d65046d666565040000000009fc026d650474666565040000006410fc026d650b6974656d46656554697073053130302c3010fc026d650b72756e654f7264657249642435623335343431612d323937622d343562342d383237372d613330343833303938333762000000',
    inputs: [ 0 ]
  }
}
```

#### confirmPSBT

Confirm and broadcast a signed PSBT.

```typescript
async confirmPSBT(params: ConfirmPSBTParams): Promise<ConfirmPSBTResponse>
```

**Parameters:**

- `orders` (RuneOrder[]): Array of orders (from fetchQuote)
- `address` (string): Bitcoin address
- `publicKey` (string): Bitcoin public key
- `paymentAddress` (string): Payment address
- `paymentPublicKey` (string): Payment public key
- `signedPsbtBase64` (string): Signed PSBT in base64 format
- `swapId` (string): Swap ID (from getPSBT)
- `themeID?` (string): Optional widget theme ID
- `sell?` (boolean): Optional flag for sell orders (default: false)
- `runeName?` (string): Optional rune name
- `marketplaces?` (string[]): Optional list of marketplaces to use
- `rbfProtection?` (boolean): Optional RBF protection flag (default: false)
- `signedRbfPsbtBase64?` (string): Optional signed RBF protection PSBT in base64 format

**Example:**

```typescript
const confirmation = await satsTerminal.confirmPSBT({
  orders: quote.selectedOrders,
  address: "bc1...",
  publicKey: "...",
  paymentAddress: "3Pc...",
  paymentPublicKey: "...",
  signedPsbtBase64: "cHNid...",
  signedRbfPsbtBase64: "cHNid...", // Only if rbfProtection is true
  swapId: psbtResponse.swapId,
  runeName: "LOBO•THE•WOLF•PUP",
  rbfProtection: true
});
```

**Example Response:**

```javascript
{
  marketplace: 'magiceden',
  txid: '4fb3ae350ae4366687a1e069b1d4de528f3dc9dc097382de041e979a8187312b',
  rbfProtection: {
    fundsPreparationTxId: '4fb3ae350ae4366687a1e069b1d4de528f3dc9dc097382de041e979a8187312b',
    fulfillmentId: 'bd85a36b-7a54-458a-8585-e3af2b1a8fe1'
  },
  isRbfTxid: true
}
```

## Example Flow

Below is a complete example flow for buying runes with RBF protection, including waiting for user input for signed PSBTs:

```javascript
import { SatsTerminal } from 'satsterminal-sdk';
import readlineSync from 'readline-sync';

// Configuration
const CONFIG = {
  apiKey: 'your_api_key_here',
  baseURL: 'https://api.satsterminal.com/v1',
  timeout: 30000 // 30 seconds timeout
};

// Trade Parameters
const TRADE_PARAMS = {
  runeName: 'LOBO•THE•WOLF•PUP',
  address: 'bc1p...',
  publicKey: '...',
  paymentAddress: '3Pc...',
  paymentPublicKey: '...',
  btcAmount: 0.0001,
  rbfProtection: true,
  marketplaces: ["MagicEden"],
  fill: true
};

// Initialize the SDK
const satsTerminal = new SatsTerminal(CONFIG);

// Function to get user input for signed PSBT
const getSignedPSBT = () => {
  console.log('\nPlease sign the PSBT and enter the signed PSBT base64 string:');
  return readlineSync.question('Signed PSBT: ');
};

// Function to get user input for signed RBF PSBT
const getSignedRbfPSBT = () => {
  console.log('\nPlease sign the RBF protection PSBT and enter the signed RBF PSBT base64 string:');
  return readlineSync.question('Signed RBF PSBT: ');
};

// Main execution
(async () => {
  try {
    // 1. Get a quote for the trade
    console.log('\n1. Fetching quote...');
    const quote = await satsTerminal.fetchQuote({
      btcAmount: TRADE_PARAMS.btcAmount,
      runeName: TRADE_PARAMS.runeName,
      address: TRADE_PARAMS.address,
      marketplaces: TRADE_PARAMS.marketplaces,
      rbfProtection: TRADE_PARAMS.rbfProtection,
      fill: TRADE_PARAMS.fill
    });
  
    console.log('Best marketplace:', quote.bestMarketplace);
    console.log('Quote:', quote.selectedOrders);

    // 2. Create a PSBT
    console.log('\n2. Creating PSBT...');
    const psbtResponse = await satsTerminal.getPSBT({
      orders: quote.selectedOrders,
      address: TRADE_PARAMS.address,
      publicKey: TRADE_PARAMS.publicKey,
      paymentAddress: TRADE_PARAMS.paymentAddress,
      paymentPublicKey: TRADE_PARAMS.paymentPublicKey,
      utxos: [], 
      feeRate: 5, 
      runeName: TRADE_PARAMS.runeName,
      themeID: null,
      sell: false,
      slippage: 9,
      rbfProtection: TRADE_PARAMS.rbfProtection
    });
  
    console.log('PSBT created:', psbtResponse);
  
    // Wait for user to input the signed PSBT
    const signedPsbtBase64 = getSignedPSBT();
  
    // Get signed RBF PSBT if RBF protection is enabled
    let signedRbfPsbtBase64 = null;
    if (TRADE_PARAMS.rbfProtection && psbtResponse.rbfProtected.base64) {
      console.log('\nRBF Protection PSBT:', psbtResponse.rbfProtected.base64);
      signedRbfPsbtBase64 = getSignedRbfPSBT();
    }

    // 3. Confirm the PSBT after signing
    console.log('\n3. Confirming PSBT...');
    const confirmation = await satsTerminal.confirmPSBT({
      orders: quote.selectedOrders,
      address: TRADE_PARAMS.address,
      publicKey: TRADE_PARAMS.publicKey,
      paymentAddress: TRADE_PARAMS.paymentAddress,
      paymentPublicKey: TRADE_PARAMS.paymentPublicKey,
      signedRbfPsbtBase64: signedRbfPsbtBase64,
      swapId: psbtResponse.swapId,
      runeName: TRADE_PARAMS.runeName,
      rbfProtection: TRADE_PARAMS.rbfProtection,
      signedPsbtBase64: signedPsbtBase64
    });
  
    console.log('Confirmation:', confirmation);

  } catch (error) {
    console.error('Error:', error.message);
  
    // Provide more helpful information for network errors
    if (error.code === 'ECONNREFUSED') {
      console.error('Connection refused. Make sure the API server is running and accessible.');
    } else if (error.code === 'ECONNRESET' || error.message.includes('socket hang up')) {
      console.error('Connection was reset or timed out. This could be due to:');
      console.error('- Network connectivity issues');
      console.error('- Server overload or maintenance');
      console.error('- Invalid API key or authentication');
      console.error('- Request timeout (the operation might take longer than expected)');
    }
  
    if (process.env.NODE_ENV === 'development') {
      console.error('Full error:', error);
    }
  }
})();
```

## Error Handling

The SDK throws errors with descriptive messages. Always wrap API calls in try-catch blocks:

```typescript
try {
  const result = await satsTerminal.search({ rune_name: 'PEPE' });
} catch (error) {
  if (error.message.includes('API key')) {
    console.error('Invalid API key');
  } else if (error.code === 'ECONNRESET' || error.message.includes('socket hang up')) {
    console.error('Network connection issue. Please try again.');
  } else {
    console.error('An error occurred:', error);
  }
}
```

## Common Issues

### Invalid API Key

Ensure your API key is valid and properly configured. Only approved partners can use the SDK.

### Network Errors

- Check your internet connection
- Verify the API endpoint is accessible
- Check if your firewall is blocking requests
- Consider increasing the timeout value in the configuration

### PSBT Errors

- Ensure all required parameters are provided
- Ensure the PSBT is valid and signed
- Verify the format of public keys and addresses
- Check if UTXOs are valid and confirmed

### RBF Protection Issues

- Make sure both the main PSBT and RBF protection PSBT are properly signed
- Verify that the RBF protection flag is set to true in all relevant method calls
- Ensure the signed RBF PSBT is passed to the confirmPSBT method

For additional support, contact our support team.
