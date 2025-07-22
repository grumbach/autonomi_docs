# Chunks

Chunks are native data types in the Autonomi Network:

* **Size:** 4MB of raw bytes
* **Content-addressed:** Address is the hash of its content
* **Immutable & self-verifiable:** Once stored, data cannot be modified, and its integrity can be verified by computing the hash and comparing it to the address.

### Client Methods

* **chunk_get**\
  Retrieves a chunk from the network by its address.
* **chunk_put**\
  Uploads a chunk to the network with payment handling.\
  Returns the total cost and the chunk's address.
* **chunk_cost**\
  Estimates the storage cost for a chunk.

---

## Usage Examples

### Rust
```rust
use autonomi::Client;
use autonomi::client::payment::PaymentOption;
use autonomi::client::chunk::{Chunk, Bytes};
use test_utils::evm::get_funded_wallet;
use eyre::Result;

#[tokio::main]
async fn main() -> Result<()> {
    // Initialize a local client and test wallet
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();

    // Create a Chunk with some data
    let chunk = Chunk::new(Bytes::from("Hello, world!"));

    // Estimate cost
    let cost = client.chunk_cost(chunk.address()).await?;
    println!("Chunk cost: {cost}");

    // Upload chunk with payment
    let payment_option = PaymentOption::from(&wallet);
    let (put_cost, addr) = client.chunk_put(&chunk, payment_option).await?;
    assert_eq!(addr, *chunk.address());
    println!("Chunk put cost: {put_cost}");

    // Allow time for replication
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    // Retrieve and verify the chunk
    let got = client.chunk_get(&addr).await?;
    assert_eq!(got, chunk.clone());
    println!("Chunk retrieved successfully");
    Ok(())
}
```

### Python
```python
from autonomi_client import Client, PaymentOption, Chunk
import asyncio

async def main():
    client = await Client.init_local()
    # Assume you have a funded wallet object
    wallet = ...

    # Create a chunk
    chunk = Chunk(b"Hello, world!")

    # Estimate cost
    cost = await client.chunk_cost(chunk.address)
    print(f"Chunk cost: {cost}")

    # Upload chunk with payment
    payment_option = PaymentOption.wallet(wallet)
    put_cost, addr = await client.chunk_put(chunk.value, payment_option)
    print(f"Chunk put cost: {put_cost}")

    # Retrieve and verify the chunk
    got = await client.chunk_get(addr)
    assert got == chunk.value
    print("Chunk retrieved successfully")

asyncio.run(main())
```

### JavaScript/TypeScript
```js
import { Client, PaymentOption, Chunk } from '@withautonomi/autonomi';

async function main() {
    const client = await Client.initLocal();
    // Assume you have a funded wallet object
    const wallet = ...;

    // Create a chunk
    const chunk = new Chunk(Buffer.from('Hello, world!'));

    // Estimate cost
    const cost = await client.chunkCost(chunk.address());
    console.log(`Chunk cost: ${cost}`);

    // Upload chunk with payment
    const paymentOption = PaymentOption.wallet(wallet);
    const [putCost, addr] = await client.chunkPut(chunk, paymentOption);
    console.log(`Chunk put cost: ${putCost}`);

    // Retrieve and verify the chunk
    const got = await client.chunkGet(addr);
    if (Buffer.compare(got, chunk.value()) === 0) {
        console.log('Chunk retrieved successfully');
    } else {
        throw new Error('Chunk data mismatch');
    }
}

main();
```
