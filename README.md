# sui-local-net-demo
[sui-local-net-demo](https://github.com/roman1e2f5p8s/sui-local-net-demo-1) describes how to start a local [Sui](https://sui.io/) network.

## Preliminary steps
1. Update Rust and update/install Sui as described in [Sui Docs](https://docs.sui.io/guides/developer/getting-started/sui-install):
```bash
rustup update stable
cargo install --locked --git https://github.com/MystenLabs/sui.git --branch main sui
```
2. To run the local Sui network with a persisted state, generate a config to store db and genesis configs:
```bash
SUI_LOCAL_CONFIG_DIR=<some-directory>
mkdir $SUI_LOCAL_CONFIG_DIR
sui genesis -f --with-faucet --working-dir=$SUI_LOCAL_CONFIG_DIR
```
3. Clone the [Sui repo](https://github.com/MystenLabs/sui/tree/main):
```bash
git clone https://github.com/MystenLabs/sui.git && cd sui
```
4. In the root folder of the `sui` repo, install a local Sui Explorer:
```bash
sudo npm install -g pnpm
pnpm install
pnpm turbo build
```
  - If the `EHOSTUNREACH` error appears, disable IPv6, and then repeat the intallation:
    ```bash
    sudo sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
    ```
5. Start the explorer dev server:
```bash
pnpm explorer dev
```

---

## Closing steps
1. Once the local network is started using on the the methods provided below, it can be verified that it is running using a simple request:
```bash
curl --location --request POST 'http://127.0.0.1:9000' \
--header 'Content-Type: application/json' \
--data-raw '{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "sui_getTotalTransactionBlocks",
  "params": []
}'
```

2. Keeping the explorer dev server running (step 5), open the local Sui Explorer at the following URL: http://localhost:3000/.

---

## Method 1: Use `sui-test-validator`
Sui-provides the [sui-test-validator](https://github.com/MystenLabs/sui/tree/main/crates/sui-test-validator) binary, which starts a local network that includes a Sui Full node, a Sui validator, a Sui faucet and (optionally) an indexer. This method is the simplest one and recommnded in the official Sui tutorial [Connect to a Local Network](https://docs.sui.io/guides/developer/getting-started/local-network).

Nativate to the root folder of the `sui` repo cloned in step 3 and build `sui-test-validator`:
```bash
cd sui && cargo build --bin sui-test-validator
```

Start the local network using the following sommand:
```bash
RUST_LOG="off,sui_node=info" \
./target/debug/sui-test-validator --config-dir $SUI_LOCAL_CONFIG_DIR
```

In another terminal window, verify that the local network is running as described in step 1.

On the local Sui explorer, observe that the local network is running es described in step 2.


---

### Sources
- [sui-test-validator | MystenLabs/Sui](https://github.com/MystenLabs/sui/tree/main/crates/sui-test-validator)
- [Connect to a Local Network | Sui Docs](https://docs.sui.io/guides/developer/getting-started/local-network)
- [Start sui-test-validator (local network) inside docker container](https://github.com/MystenLabs/sui/issues/15279)
- [sui-indexer | MystenLabs/Sui](https://github.com/MystenLabs/sui/tree/main/crates/sui-indexer)
- [Sui Indexer | Sui Docs](https://docs.sui.io/concepts/sui-architecture/indexer-functions#:~:text=Sui%20Indexer%20is%20an%20off,from%20chain%20and%20derivative%20data.)
