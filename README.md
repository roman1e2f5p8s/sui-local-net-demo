# sui-local-net-demo
[sui-local-net-demo](https://github.com/roman1e2f5p8s/sui-local-net-demo) demonstrates how to start a local [Sui](https://sui.io/) network on a physical machine.

## Preliminary steps
Preliminary steps include installing dependencies and generating configs.

1. Update Rust and update/install Sui as described in [Sui Docs](https://docs.sui.io/guides/developer/getting-started/sui-install):
```bash
rustup update stable
cargo install --locked --git https://github.com/MystenLabs/sui.git --branch main sui
```

2. If you would like to run the local Sui network with a persisted state, generate a config to store db and genesis configs:
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

5. Start the Sui Explorer dev server:
```bash
pnpm explorer dev
```

---

## Closing steps
Closing steps are meant to be taken after the local Sui network is started to verify that it is running.

1. Once the local network is started using one of the methods provided below, it can be verified that it is running using a simple request:
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

2. Keeping the explorer dev server running (step 5 in [Preliminary steps](#preliminary-steps)), open the local Sui Explorer at the following URL: http://localhost:3000/.

---

## Method 1: Use `sui-test-validator`
Sui-provides the [sui-test-validator](https://github.com/MystenLabs/sui/tree/main/crates/sui-test-validator) binary that starts a local network that includes a Sui Full node, a Sui validator, a Sui faucet and (optionally) an indexer. This method is the simplest one and it is recommnded in the official Sui tutorial [Connect to a Local Network](https://docs.sui.io/guides/developer/getting-started/local-network).

Nativate to the root folder of the `sui` repo (cloned in step 3 of [Preliminary steps](#preliminary-steps)) and build `sui-test-validator`:
```bash
cd sui && cargo build --bin sui-test-validator
```

Start the local network using the following sommand:
```bash
RUST_LOG="off,sui_node=info" ./target/debug/sui-test-validator
```

To start the local Sui network with a persisted state (make sure you completed step 2 of [Preliminary steps](#preliminary-steps)), use the following sommand:
```bash
RUST_LOG="off,sui_node=info" \
  ./target/debug/sui-test-validator --config-dir $SUI_LOCAL_CONFIG_DIR
```

In another terminal window, verify that the local network is running as described in step 1 of [Closing steps](#closing-steps).

On the local Sui explorer, observe that the local network is running es described in step 2 of [Closing steps](#closing-steps).

---

## Method 2: Use `sui start`
`sui start` starts a local Sui network by running pre-generated genesis configs.

Make sure you completed step 2 of [Preliminary steps](#preliminary-steps) as it is required to generate genesis configs in this method.

Start the local Sui network using the following command:
```bash
RUST_LOG="off,sui_node=info" \
  sui start --network.config=$SUI_LOCAL_CONFIG_DIR/network.yaml
```

In another terminal window, verify that the local network is running as described in step 1 of [Closing steps](#closing-steps).

On the local Sui explorer, observe that the local network is running es described in step 2 of [Closing steps](#closing-steps).

---

## Method 3: Use `sui genesis-ceremony`
[sui genesis-ceremony](https://github.com/MystenLabs/sui/blob/main/crates/sui/genesis.md) orchestrates a Sui Genesis Ceremony.

1. Create a new empty [GitHub](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository) repo. Assume it is named `sui-genesis`.

2. Clone the repo on a local machine and navigate to its root folder:
```bash
git clone https://github.com/<owner>/sui-genesis.git && cd sui-genesis
```

3. Initialize a Sui genesis ceremony and push the changes to the repo:
```bash
sui genesis-ceremony init
git add .
git commit -S -m "init genesis"
git push -u origin main
```

4. Generate a protocol key (see [Sui Node Key Management](https://github.com/MystenLabs/sui/blob/main/nre/sui_for_node_operators.md#key-management) for detail):
```bash
k=$(sui keytool generate bls12381 | grep suiAddress | awk '{print $4}'); \
  mv bls-$k.key val_0.key
```

5. Generate a worker key:
```bash
k=$(sui keytool generate ed25519 | grep suiAddress | awk '{print $4}'); \
  mv $k.key val_0_worker.key
```

6. Generate an account key:
```bash
k=$(sui keytool generate ed25519 | grep suiAddress | awk '{print $4}'); \
  mv $k.key val_0_account.key
```

7. Generate a network key:
```bash
k=$(sui keytool generate ed25519 | grep suiAddress | awk '{print $4}'); \
  mv $k.key val_0_network.key
```

8. Add validator information:
```bash
sui genesis-ceremony add-validator \
  --name validator-0 \
  --validator-key-file val_0.key \
  --worker-key-file val_0_worker.key \
  --account-key-file val_0_account.key \
  --network-key-file val_0_network.key \
  --network-address /ip4/0.0.0.0/tcp/8080/http \
  --p2p-address /ip4/0.0.0.0/tcp/8084/http \
  --narwhal-primary-address /ip4/0.0.0.0/tcp/8081/http \
  --narwhal-worker-address /ip4/0.0.0.0/tcp/8082/http \
  --description "Local Sui validator 0" \
  --image-url https://github.com/MystenLabs/sui/blob/main/docs/site/static/img/logo.svg \
  --project-url https://github.com/roman1e2f5p8s/sui-local-net-demo
```
Values for arguments `image-url` and `project-url` can be specified as empty strings.

9. Push the changes to the repo:
```bash
git add .
git commit -S -m "add validator-0 information"
git push -u origin main
```

---

### Sources
- [sui-test-validator | MystenLabs/Sui](https://github.com/MystenLabs/sui/tree/main/crates/sui-test-validator)
- [Connect to a Local Network | Sui Docs](https://docs.sui.io/guides/developer/getting-started/local-network)
- [Start sui-test-validator (local network) inside docker container](https://github.com/MystenLabs/sui/issues/15279)
- [sui-indexer | MystenLabs/Sui](https://github.com/MystenLabs/sui/tree/main/crates/sui-indexer)
- [Sui Indexer | Sui Docs](https://docs.sui.io/concepts/sui-architecture/indexer-functions#:~:text=Sui%20Indexer%20is%20an%20off,from%20chain%20and%20derivative%20data.)
- [Genesis Ceremony | MystenLabs/Sui](https://github.com/MystenLabs/sui/blob/main/crates/sui/genesis.md)
- [Sui for Node Operators | MystenLabs/Sui](https://github.com/MystenLabs/sui/blob/main/nre/sui_for_node_operators.md)
