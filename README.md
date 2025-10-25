

#  GamingForge -  Gaming Platform Smart Contract

**Version:** 1.0
**Language:** [Clarity](https://docs.stacks.co/write-smart-contracts/clarity-overview)
**Author:** Gamin Dev Team
**License:** MIT

---

## 📘 Overview

The **Gamin Gaming Platform Smart Contract** provides a decentralized system for **managing in-game digital assets**, including **ownership, minting, trading, and player progression tracking**.

It supports **batch operations** for scalability and efficiency, while ensuring **strong access control, validation, and security** across all functions.

This contract can serve as the backbone for blockchain-based gaming ecosystems, NFT-driven games, or metaverse-style economies where players can **own, trade, and evolve their in-game items**.

---

## 🚀 Key Features

* **Asset Ownership & Metadata**

  * Each in-game asset is a unique, verifiable entity tied to a player.
  * Metadata URIs point to off-chain asset data (e.g., NFT art, stats, skins, etc.).
  * Assets can be transferable or non-transferable (e.g., soulbound items).

* **Batch Operations**

  * Batch minting and batch transferring to minimize transaction costs and gas fees.
  * Batch size limited (`max-batch-size = 10`) to prevent performance issues.

* **Marketplace Functionality**

  * Players can list assets for sale with a price in STX.
  * Other players can purchase listed assets via `stx-transfer?`.
  * Sellers can delist their assets anytime.

* **Player Stats Tracking**

  * Player stats include **experience** and **level**.
  * Configurable limits:

    * `max-level = 100`
    * `max-experience = 10,000`

* **Strong Validation and Error Handling**

  * Error codes for invalid inputs, unauthorized actions, and ownership mismatches.
  * Checks for metadata length, ownership, and transferability.

---

## ⚙️ Technical Details

| Variable / Constant    | Type        | Description                                               |
| ---------------------- | ----------- | --------------------------------------------------------- |
| `contract-owner`       | `principal` | Address of the contract deployer.                         |
| `assets`               | `map`       | Stores each asset’s owner, metadata, and transferability. |
| `asset-prices`         | `map`       | (Reserved) for future asset pricing system.               |
| `player-stats`         | `map`       | Tracks experience and level of each player.               |
| `marketplace-listings` | `map`       | Active listings of assets for sale.                       |
| `asset-counter`        | `uint`      | Tracks total assets minted.                               |

---

## 🧩 Core Functionalities

### 🔹 Asset Management

#### **Mint Single Asset**

```clarity
(mint-asset (metadata-uri (string-utf8 256)) (transferable bool)) → (response uint uint)
```

* Only the contract owner can mint.
* Validates metadata length.
* Increments asset counter and assigns ownership to the contract owner.

#### **Batch Minting**

```clarity
(batch-mint-assets 
    (metadata-uris (list 10 (string-utf8 256))) 
    (transferable-list (list 10 bool))) → (response (list 10 uint) uint)
```

* Mints multiple assets in one transaction.
* Each metadata URI corresponds to a transferability flag.

#### **Transfer Asset**

```clarity
(transfer-asset (asset-id uint) (recipient principal)) → (response bool uint)
```

* Only the asset owner can transfer.
* Asset must be transferable.
* Prevents self-transfer.

#### **Batch Transfer**

```clarity
(batch-transfer-assets 
    (asset-ids (list 10 uint)) 
    (recipients (list 10 principal))) → (response (list 10 bool) uint)
```

* Transfers multiple assets to multiple recipients.

---

### 🔹 Marketplace Operations

#### **List Asset for Sale**

```clarity
(list-asset-for-sale (asset-id uint) (price uint)) → (response bool uint)
```

* Owner lists an asset with a STX price.
* Asset must be transferable.
* Stores listing data: seller, price, block height.

#### **Purchase Asset**

```clarity
(purchase-asset (asset-id uint)) → (response bool uint)
```

* Buyer purchases a listed asset.
* Transfers STX to seller and updates ownership.

#### **Delist Asset**

```clarity
(delist-asset (asset-id uint)) → (response bool uint)
```

* Only the seller can delist.
* Removes the asset from the marketplace.

---

### 🔹 Player Stats

#### **Update Player Stats**

```clarity
(update-player-stats (experience uint) (level uint)) → (response bool uint)
```

* Any player can update their own stats.
* Enforces max limits for `experience` and `level`.

---

### 🔹 Read-only Queries

#### Get Asset Details

```clarity
(get-asset-details (asset-id uint)) → (optional { owner: principal, metadata-uri: (string-utf8 256), transferable: bool })
```

#### Get Marketplace Listing

```clarity
(get-marketplace-listing (asset-id uint)) → (optional { seller: principal, price: uint, listed-at: uint })
```

#### Get Player Stats

```clarity
(get-player-stats (player principal)) → (optional { experience: uint, level: uint })
```

#### Get Total Assets

```clarity
(get-total-assets) → uint
```

---

## ⚠️ Error Codes

| Error                | Code   | Meaning                              |
| -------------------- | ------ | ------------------------------------ |
| `err-owner-only`     | `u100` | Action restricted to contract owner. |
| `err-not-found`      | `u101` | Asset or listing not found.          |
| `err-not-authorized` | `u102` | Caller not authorized.               |
| `err-invalid-input`  | `u103` | Input parameters invalid.            |
| `err-invalid-price`  | `u104` | Invalid or zero price.               |

---

## 🔐 Security & Design Considerations

* **Ownership Enforcement:**
  All write operations validate ownership (`tx-sender` == asset owner).

* **Transfer Restrictions:**
  Non-transferable assets (e.g., achievements or rewards) are protected.

* **Batch Operation Limits:**
  Prevents denial-of-service or excessive gas use.

* **Price Validation:**
  Prevents zero-price listings to avoid spam or abuse.

* **Error Handling:**
  Comprehensive error codes with `asserts!` and `unwrap!` ensure predictable behavior.

---

## 🧠 Example Workflow

### 1. Mint a new item

```clarity
(contract-call? .gamin-platform mint-asset "ipfs://item-metadata.json" true)
```

### 2. List it for sale

```clarity
(contract-call? .gamin-platform list-asset-for-sale u1 u1000)
```

### 3. Purchase it

```clarity
(contract-call? .gamin-platform purchase-asset u1)
```

### 4. Update player stats

```clarity
(contract-call? .gamin-platform update-player-stats u5000 u20)
```

### 5. Query asset details

```clarity
(contract-call? .gamin-platform get-asset-details u1)
```

---

## 🧾 Future Enhancements

* Integrate **royalty distribution** for creators.
* Add **auction system** for dynamic pricing.
* Implement **asset fusion or upgrading** logic.
* Add **off-chain metadata verification** hooks.

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.
