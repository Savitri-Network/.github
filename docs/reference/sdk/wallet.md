# Wallet

The `Wallet` manages ed25519 key pairs, signs messages, and optionally connects to an RPC node for on-chain queries.

## Key Management

### Create a New Wallet

```rust
use savitri_sdk::Wallet;

// Random keypair (cryptographically secure)
let wallet = Wallet::new();
println!("Address: {}", wallet.address());  // 64 hex chars (32-byte pubkey)
```

### Import from Private Key

```rust
// From raw bytes
let private_key: [u8; 32] = [/* your key */];
let wallet = Wallet::from_private_key(&private_key)?;

// From hex string
let wallet = Wallet::from_private_key_hex("aabbccdd...64hexchars")?;

// With 0x prefix
let wallet = Wallet::from_private_key_hex("0xaabbccdd...64hexchars")?;
```

### Export Keys

```rust
// Public key (safe to share)
let pubkey = wallet.public_key();
println!("Public key: {}", hex::encode(pubkey.as_bytes()));

// Address (derived from public key)
println!("Address: {}", wallet.address());

// Private key (NEVER share or log)
let privkey: [u8; 32] = wallet.private_key();
```

## Signing

### Sign a Message

```rust
let message = b"Hello, Savitri!";
let signature: [u8; 64] = wallet.sign_message(message);
```

### Verify a Signature

```rust
wallet.verify_signature(message, &signature)?;
// Returns Ok(()) if valid, Err if invalid
```

The signing scheme uses Ed25519 (ed25519-dalek). For transaction signing, the message is first SHA-256 hashed:

```
message = from_hex || to_hex || amount_le_u64 || nonce_le_u64 || fee_le_u128
hash    = SHA-256(message)
sig     = Ed25519_sign(hash)
```

## RPC Integration

Connect a wallet to a node for on-chain queries:

```rust
// Create with RPC connection
let wallet = Wallet::with_rpc("http://localhost:8545")?;

// Or connect later
let mut wallet = Wallet::new();
wallet.connect_rpc("http://localhost:8545")?;

// Query balance (requires RPC)
let balance: String = wallet.get_balance().await?;
println!("Balance: {}", balance);

// Query nonce (requires RPC)
let nonce: u64 = wallet.get_nonce().await?;
println!("Nonce: {}", nonce);

// Access underlying RPC client
if let Some(rpc) = wallet.rpc() {
    let height = rpc.get_block_number().await?;
    println!("Height: {}", height);
}
```

## Security

- **Zeroization**: Private key material is zeroized when the wallet is dropped (`Drop` trait via `zeroize` crate).
- **Clone behavior**: Cloned wallets do NOT inherit the RPC connection (they start disconnected).
- **No persistence**: The wallet does not save keys to disk. Use your own secure storage.

```rust
{
    let wallet = Wallet::new();
    // ... use wallet ...
} // wallet dropped here — private key bytes are zeroed in memory
```
