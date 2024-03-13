# sui-local-net-demo-1
[sui-local-net-demo-1]() describes how to start a local [Sui](https://sui.io/) network using the [sui-test-validator](https://github.com/MystenLabs/sui/tree/main/crates/sui-test-validator) binary provided by Sui.

The [`sui-test-validator`](https://github.com/MystenLabs/sui/tree/main/crates/sui-test-validator) starts a local network that includes a Sui Full node, a Sui validator, a Sui faucet and (optionally) an indexer.


1. [Install Sui](https://docs.sui.io/guides/developer/getting-started/sui-install):
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
3. Clone the Sui repo and build `sui-test-validator`:
```bash
git clone https://github.com/MystenLabs/sui.git && cd sui
cargo build --bin sui-test-validator
```
4. Start the local network:
```bash
RUST_LOG="off,sui_node=info" ./target/debug/sui-test-validator --config-dir $SUI_LOCAL_CONFIG_DIR
```
5. Verify that the local network is running using a simple request:
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
6. Install Sui Explorer and Sui Wallet locally:
    - Install [pnpm](https://pnpm.io/installation): `sudo npm install -g pnpm`
    - In the `sui` root folder (step 3), install dependencies: `pnpm install`
    - If the `EHOSTUNREACH` error appears, disable IPv6: `sudo sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1` and repeat `pnpm install`
    - Install Sui Explorer and Sui Wallet: `pnpm turbo build`
7. Start the explorer dev server: `pnpm explorer dev`
8. Open the local Sui Explorer at the following URL: http://localhost:3000

### Sources
- [sui-test-validator | MystenLabs/Sui](https://github.com/MystenLabs/sui/tree/main/crates/sui-test-validator)
- [Connect to a Local Network | Sui Docs](https://docs.sui.io/guides/developer/getting-started/local-network)
- [Start sui-test-validator (local network) inside docker container](https://github.com/MystenLabs/sui/issues/15279)
- [sui-indexer | MystenLabs/Sui](https://github.com/MystenLabs/sui/tree/main/crates/sui-indexer)
- [Sui Indexer | Sui Docs](https://docs.sui.io/concepts/sui-architecture/indexer-functions#:~:text=Sui%20Indexer%20is%20an%20off,from%20chain%20and%20derivative%20data.)
