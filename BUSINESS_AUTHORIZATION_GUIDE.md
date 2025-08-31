# Business Authorization Guide

This guide explains how to authorize a business wallet for on-chain QR code generation in the QRCodeValidator contract.

## Overview

The QRCodeValidator contract requires businesses to be explicitly authorized before they can generate QR codes on-chain. This is a security measure to prevent unauthorized QR code generation.

## Current Status

- **Contract Address (Fuji):** `0x2F2F42864Be17A46793Df0a324BBdE66093901de`
- **Contract Owner:** The deployer wallet (has admin privileges)
- **Authorization Required:** Yes, for all business wallets

## Method 1: Using the Authorization Script (Recommended)

### Prerequisites
- Access to the contract owner wallet
- Node.js and Hardhat environment set up
- Fuji testnet AVAX for gas fees

### Steps

1. **Get the business wallet address** that needs authorization
2. **Run the authorization script:**

```bash
cd pox
npx hardhat run scripts/authorize-business.ts --network fuji
```

Or with a specific business wallet:

```bash
BUSINESS_WALLET=0xYourBusinessWalletAddress npx hardhat run scripts/authorize-business.ts --network fuji
```

Or as a command line argument:

```bash
npx hardhat run scripts/authorize-business.ts --network fuji 0xYourBusinessWalletAddress
```

### Script Features
- ✅ Checks if you're the contract owner
- ✅ Verifies current authorization status
- ✅ Authorizes the business wallet
- ✅ Confirms successful authorization
- ✅ Provides helpful error messages

## Method 2: Manual Contract Interaction

### Using Hardhat Console

```bash
cd pox
npx hardhat console --network fuji
```

```javascript
// Get the contract
const QRCodeValidator = await ethers.getContractFactory("QRCodeValidator");
const contract = QRCodeValidator.attach("0x2F2F42864Be17A46793Df0a324BBdE66093901de");

// Check current owner
const owner = await contract.owner();
console.log("Owner:", owner);

// Authorize a business (only owner can do this)
const businessWallet = "0xYourBusinessWalletAddress";
const tx = await contract.authorizeBusiness(businessWallet, true);
await tx.wait();

// Verify authorization
const isAuthorized = await contract.authorizedBusinesses(businessWallet);
console.log("Is authorized:", isAuthorized);
```

### Using Etherscan/Snowtrace

1. Go to the contract on [Snowtrace](https://testnet.snowtrace.io/address/0x2F2F42864Be17A46793Df0a324BBdE66093901de)
2. Connect your wallet (must be the owner)
3. Go to "Write Contract" tab
4. Find `authorizeBusiness` function
5. Enter:
   - `business`: The wallet address to authorize
   - `authorized`: `true`
6. Execute the transaction

## Method 3: Using the UI Request Feature

The business dashboard now includes a "Request Authorization" button that:
- Shows your wallet address
- Pre-fills an email template
- Provides all necessary information for the admin

## Verification

After authorization, you can verify it worked by:

1. **Check the UI:** The QR generator should show "On-Chain QR Generation Enabled"
2. **Check contract state:**
```javascript
const isAuthorized = await contract.authorizedBusinesses("0xYourWalletAddress");
console.log(isAuthorized); // Should be true
```

## Troubleshooting

### "Ownable: caller is not the owner"
- Only the contract owner can authorize businesses
- Make sure you're using the correct owner wallet
- Check the owner with: `await contract.owner()`

### "Transaction failed"
- Ensure you have enough AVAX for gas fees
- Check that the wallet address is valid
- Verify you're on the correct network (Fuji testnet)

### UI still shows "Offline Mode"
- Refresh the page after authorization
- Check that the wallet address matches exactly
- Verify the authorization was successful on-chain

## Contract Functions

### For Owner Only
- `authorizeBusiness(address business, bool authorized)` - Authorize/deauthorize a business
- `pause()` / `unpause()` - Pause/unpause the contract

### For Authorized Businesses
- `generateQRCode(string businessName, string location, uint8 category, uint256 customExpiry)` - Generate QR codes

### For Everyone
- `checkQRCode(bytes32 qrHash)` - Check if a QR code is valid
- `authorizedBusinesses(address)` - Check if a business is authorized

## Security Notes

- Only the contract owner can authorize businesses
- Authorization is required for on-chain QR generation
- Unauthorized businesses can still use offline mode
- The contract is upgradeable (UUPS pattern)

## Support

If you need help with authorization:
1. Use the "Request Authorization" button in the UI
2. Contact the platform administrator
3. Provide your wallet address and business details

