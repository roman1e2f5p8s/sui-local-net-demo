# sui-local-net-demo
[sui-local-net-demo](https://github.com/roman1e2f5p8s/sui-local-net-demo) demonstrates how to start a local [Sui](https://sui.io/) network and run a Sui validator node using `sui genesis-ceremony` and `sui-node` on a local machine.

> [!NOTE]
> This demo was tested using:
> - Rust: `rustc 1.76.0 (07dca489a 2024-02-04)`
> - Sui:  `sui 1.21.0-473e4e256`

## Preliminary steps
Preliminary steps include installing dependencies and generating configs.

*1. Update Rust and update/install Sui as described in [Sui Docs](https://docs.sui.io/guides/developer/getting-started/sui-install):*
```bash
rustup update stable
cargo install --locked --git https://github.com/MystenLabs/sui.git --branch main sui
```
<!-- ![Rust and Sui version](/images/rust-sui-version.png) -->

*2. If you would like to run a local Sui network with a persisted state, generate a config to store db and genesis configs:*
```bash
SUI_LOCAL_CONFIG_DIR=<some-directory>
mkdir $SUI_LOCAL_CONFIG_DIR
sui genesis -f --with-faucet --working-dir=$SUI_LOCAL_CONFIG_DIR
```
<!-- The expected output and the list of generated files should be as follows: -->
<!-- ![sui genesis](/images/sui-genesis.png) -->

*3. Clone the [Sui repo](https://github.com/MystenLabs/sui/tree/main):*
```bash
git clone https://github.com/MystenLabs/sui.git && cd sui
```
<!-- ![git clone sui](/images/clone-sui.png) -->

*4. In the root folder of the `sui` repo, install a local Sui Explorer:*
```bash
sudo npm install -g pnpm
pnpm install
pnpm turbo build
```
<!-- ![pnpm install](/images/pnpm-install.png) -->
  - If the `EHOSTUNREACH` error appears, disable IPv6, and then repeat the installation:
    ```bash
    sudo sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
    ```

*5. Start the Sui Explorer dev server by running the following command in the root folder of the `sui` repo:*

```bash
pnpm explorer dev
```
![run sui explorer](/images/run-sui-explorer.png)
- If you open the local Sui Explorer at http://localhost:3000/ and it redirects to the following
  ![sui explorer redirect](/images/sui-explorer-redirect.png)
  you will need to modify the following line in `apps/explorer/src/components/Layout/PageLayout.tsx` in the root folder of the `sui` repo:
  ```typescript
  const enableExplorerRedirect = useFeatureIsOn('explorer-redirect');
  ```
  to
  ```typescript
  const enableExplorerRedirect = false; //useFeatureIsOn('explorer-redirect');
  ```
  (there is no need to repeat steps 4-5 of this section after the modification is made).
  The local Sui Explorer at http://localhost:3000/ should then look as follows:
  ![sui explorer 0](/images/sui-explorer-0.png)

---

## Closing steps
Closing steps are meant to be taken after the local Sui network is started, to verify that it is running.

*1. Once the local network is started using one of the methods provided below, it can be verified that it is running using a simple request:*
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

*2. Keeping the explorer dev server running (step 5 in [Preliminary steps](#preliminary-steps)), open the local Sui Explorer at the following URL: http://localhost:3000/.*

---

## Method 1: Use `sui-test-validator`
Sui-provides the [sui-test-validator](https://github.com/MystenLabs/sui/tree/main/crates/sui-test-validator) binary that starts a local network that includes a Sui Full node, a Sui validator, a Sui faucet and (optionally) an indexer. This method is the simplest one and it is recommended in the official Sui tutorial [Connect to a Local Network](https://docs.sui.io/guides/developer/getting-started/local-network).

*1. Navigate to the root folder of the `sui` repo (cloned in step 3 of [Preliminary steps](#preliminary-steps)) and build `sui-test-validator`:*
```bash
cd sui && cargo build --bin sui-test-validator
```
<!-- ![build test validator](/images/build-test-validator.png) -->

*2. Start the local network using the following command:*
```bash
RUST_LOG="off,sui_node=info" ./target/debug/sui-test-validator
```
![run test validator](/images/run-test-validator.png)

*3. To start the local Sui network with a persisted state (make sure you completed step 2 of [Preliminary steps](#preliminary-steps)), use the following command:*
```bash
RUST_LOG="off,sui_node=info" \
  ./target/debug/sui-test-validator --config-dir $SUI_LOCAL_CONFIG_DIR
```
![run test validator with config](/images/run-test-validator-with-config.png)

*4. In another terminal window, verify that the local network is running as described in step 1 of [Closing steps](#closing-steps).*
![test validator request](/images/test-validator-request.png)

*5. On the local Sui explorer, observe that the local network is running es described in step 2 of [Closing steps](#closing-steps).*
![test validator explorer](/images/test-validator-explorer.png)

---

## Method 2: Use `sui start`
`sui start` starts a local Sui network by running pre-generated genesis configs.

*1. Make sure you completed step 2 of [Preliminary steps](#preliminary-steps) as it is required to generate genesis configs in this method.*

*2. Start the local Sui network using the following command:*
```bash
RUST_LOG="off,sui_node=info" \
  sui start --network.config=$SUI_LOCAL_CONFIG_DIR/network.yaml
```
![run sui start](/images/run-sui-start.png)

*3. In another terminal window, verify that the local network is running as described in step 1 of [Closing steps](#closing-steps).*
![sui start request](/images/sui-start-request.png)

*4. On the local Sui explorer, observe that the local network is running es described in step 2 of [Closing steps](#closing-steps).*
![sui start explorer](/images/sui-start-explorer.png)

---

## Run a Sui validator node using `sui genesis-ceremony` and `sui-node`

### Generate `genesis.blob`
[sui genesis-ceremony](https://github.com/MystenLabs/sui/blob/main/crates/sui/genesis.md) orchestrates a Sui Genesis Ceremony.

> [!IMPORTANT]
> Steps 1-3, 10 should be performed only by the master of ceremony.
> 
> Steps 2, 4-9, 11 should be performed by each participating validator.

*1. Create a new empty [GitHub](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository) repo. Assume it is named `sui-genesis-demo`.*
<!-- ![create sui genesis demo](/images/create-sui-genesis-demo.png) -->

*2. Clone the repo on a local machine and navigate to its root folder:*
```bash
git clone https://github.com/<owner>/sui-genesis-demo.git && cd sui-genesis-demo
```
<!-- ![clone sui genesis demo](/images/clone-sui-genesis-demo.png) -->

*3. Initialize a Sui genesis ceremony and push the changes to the repo:*
```bash
sui genesis-ceremony init
git add .
git commit -S -m "init genesis"
git push -u origin main
```
![sui genesis init](/images/sui-genesis-init.png)

*4. Generate a protocol key (see [Sui Node Key Management](https://github.com/MystenLabs/sui/blob/main/nre/sui_for_node_operators.md#key-management) for detail):*
```bash
k=$(sui keytool generate bls12381 | grep suiAddress | awk '{print $4}'); \
  mv bls-$k.key val_0.key
```

*5. Generate a worker key:*
```bash
k=$(sui keytool generate ed25519 | grep suiAddress | awk '{print $4}'); \
  mv $k.key val_0_worker.key
```

*6. Generate an account key:*
```bash
k=$(sui keytool generate ed25519 | grep suiAddress | awk '{print $4}'); \
  mv $k.key val_0_account.key
```

*7. Generate a network key:*
```bash
k=$(sui keytool generate ed25519 | grep suiAddress | awk '{print $4}'); \
  mv $k.key val_0_network.key
```
![sui genesis generate keys](/images/sui-genesis-gen-keys.png)

*8. Add validator information:*
```bash
sui genesis-ceremony add-validator \
  --name validator-0 \
  --validator-key-file val_0.key \
  --worker-key-file val_0_worker.key \
  --account-key-file val_0_account.key \
  --network-key-file val_0_network.key \
  --network-address /ip4/0.0.0.0/tcp/8080/http \
  --narwhal-primary-address /ip4/0.0.0.0/udp/8081/http \
  --narwhal-worker-address /ip4/0.0.0.0/udp/8082/http \
  --p2p-address /ip4/0.0.0.0/udp/8084/http \
  --description "Local Sui validator 0" \
  --image-url https://github.com/MystenLabs/sui/blob/main/docs/site/static/img/logo.svg \
  --project-url https://github.com/roman1e2f5p8s/sui-local-net-demo
```
![add validator information](/images/add-validator-info.png)
Information about addresses can be found in [Connectivity ports](https://github.com/MystenLabs/sui/blob/main/nre/sui_for_node_operators.md#connectivity).
Values for arguments `image-url` and `project-url` can be specified as empty strings.

*9. Push the changes to the repo:*
```bash
git add .
git commit -S -m "add validator-0 information"
git push -u origin main
```

*10. Once all validators have been added, the master of ceremony needs to build the genesis object and push the changes to the repo:*
```bash
sui genesis-ceremony build-unsigned-checkpoint
git add .
git commit -S -m "build genesis"
git push -u origin main
```
![build unsigned checkpoint](/images/build-unsigned-checkpoint.png)

*11. Each validator will then need to verify and sign genesis and push the changes to the repo:*
```bash
sui genesis-ceremony verify-and-sign --key-file val_0.key
git add .
git commit -S -m "verify and sign genesis"
git push -u origin main
```
![verify and sign](/images/verify-and-sign.png)

*12. Once all validators have successfully verified and signed genesis, the master of ceremony needs to finalize the ceremony and then the genesis state can be distributed:*
```bash
sui genesis-ceremony finalize
git add .
git commit -S -m "finalize genesis"
git push -u origin main
```
![finalize genesis](/images/finalize-genesis.png)

For this demonstration, we used the [sui-genesis-demo](https://github.com/roman1e2f5p8s/sui-genesis-demo) repo, where you can find all files generated during the ceremony.

### Run a Sui node using `systemd`
Follow these steps to setup a Sui node as a systemd service. See [Run a Sui Node using Systemd | MystenLabs/Sui](https://github.com/MystenLabs/sui/blob/main/nre/systemd/README.md) for detail.

*1. Add a `sui` user and setup the `/opt/sui` directory as follows:*
```bash
sudo useradd sui
sudo mkdir -p /opt/sui/bin
sudo mkdir -p /opt/sui/config
sudo mkdir -p /opt/sui/db
sudo mkdir -p /opt/sui/key-pairs
sudo chown -R sui:sui /opt/sui
```

*2. Navigate to the root folder of the `sui` repo (cloned in step 3 of [Preliminary steps](#preliminary-steps)), build `sui-node`, and copy the built binary into `/opt/sui/bin/`:*
```bash
cd sui && git checkout $SUI_SHA
cargo build --release --bin sui-node
sudo cp ./target/release/sui-node /opt/sui/bin/
```
<!-- ![build sui node](/images/build-sui-node.png) -->

*3. Navigate to the `sui-genesis-demo` repo (see step 2 of [Generate `genesis.blob`](#generate-genesisblob)) and copy key-pairs generated in steps 4-7 of [Generate `genesis.blob`](#generate-genesisblob) into `/opt/sui/key-pairs/` as follows:*
```bash
cd sui-genesis-demo
sudo cp val_0.key /opt/sui/key-pairs/protocol.key
sudo cp val_0_worker.key /opt/sui/key-pairs/worker.key
sudo cp val_0_network.key /opt/sui/key-pairs/network.key
sudo cp val_0_account.key /opt/sui/key-pairs/account.key
```
<!-- ![copy key pairs](/images/copy-key-pairs.png) -->

*4. Download the node configuration file provided by Sui from [here](https://github.com/MystenLabs/sui/blob/main/nre/config/validator.yaml) and place it into the `/opt/sui/config/` directory:*
```bash
sudo mv validator.yaml /opt/sui/config/
```

*5. Open `/opt/sui/config/validator.yaml` and modify the following line:*
```yaml
  external-address: /dns/$HOSTNAME/udp/8084 # UPDATE THIS
```
to
```yaml
  external-address: /ip4/0.0.0.0/udp/8084/http
```
and save the file.

*6. Navigate to the `sui-genesis-demo` repo (see step 2 of [Generate `genesis.blob`](#generate-genesisblob)) and copy `genesis.blob` into `/opt/sui/config/`:*
```bash
cd sui-genesis-demo && sudo cp genesis.blob /opt/sui/config/
```

*7. After copying and creating files in `/opt/sui`, make sure they retain `sui` user permissions by re-running*
```bash
sudo chown -R sui:sui /opt/sui
```

*8. Download the sui-node systemd service unit file provided by Sui from [here](https://github.com/MystenLabs/sui/blob/main/nre/systemd/sui-node.service) and place it into the `/etc/systemd/system/` directory:*
```bash
sudo mv sui-node.service /etc/systemd/system/
```

*9. Reload systemd and enable this new service unit file:*
```bash
sudo systemctl daemon-reload
sudo systemctl enable sui-node.service
```
<!-- ![enable sui node service](/images/enable-sui-node-service.png) -->

*10. Start the validator node:*
```bash
sudo systemctl start sui-node
```
- Check that the node is up and running:
  ```bash
  sudo systemctl status sui-node
  ```
  ![sui node running](/images/sui-node-running.png)
- The logs can be shown by running:
  ```bash
  journalctl -u sui-node -f
  ```
  <!-- ![sui node logs](/images/sui-node-logs.png) -->
- To stop sui-node systemd service, run:
  ```bash
  sudo systemctl stop sui-node
  ```
  <!-- ![sui node stop](/images/sui-node-stop.png) -->
- To delete the local Sui node databases, run:
  ```bash
  sudo rm -rf /opt/sui/db/*
  ```
  <!-- ![sui node disable](/images/sui-node-disable.png) -->

---

### Sources
- [sui-test-validator | MystenLabs/Sui](https://github.com/MystenLabs/sui/tree/main/crates/sui-test-validator)
- [Connect to a Local Network | Sui Docs](https://docs.sui.io/guides/developer/getting-started/local-network)
- [Start sui-test-validator (local network) inside docker container | Issues](https://github.com/MystenLabs/sui/issues/15279)
- [Genesis Ceremony | MystenLabs/Sui](https://github.com/MystenLabs/sui/blob/main/crates/sui/genesis.md)
- [Sui for Node Operators | MystenLabs/Sui](https://github.com/MystenLabs/sui/blob/main/nre/sui_for_node_operators.md)
- [Run a Sui Node using Systemd | MystenLabs/Sui](https://github.com/MystenLabs/sui/blob/main/nre/systemd/README.md)
