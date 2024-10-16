# What does the setup do ?

It provides a way to run a co-processor, gateway and kms (in centralized mode)
using docker compose.

The pre-computed contract and account addresses/private keys are configured
across all the components for this test setup.

# How the repository is organized ?

- The docker compose file for KMS is defined
  [locally](./docker-compose/docker-compose-full.yml).

- Co-processor repository is cloned to work_dir and the Makefile is used for
  bringing up co-processor network.

- Solidity repository is cloned and used from source to funding test accounts,
  deploy contracts and run tests.

- User is expected to run gateway from source.


## Steps to run the setup

1. Run the KMS in centralized mode (including the deployment of contracts on
   KMS blockchain).

```bash
make run-full
```

2. Run the fhevm coprocessor network (including a geth node).

```bash
make run-coprocessor
```

3. Fund test accounts and deploy the fhevm solidity contracts.

```bash
make prepare-e2e-test
```

If prompted to install npm dependencies, enter `y`.

4. In a separate terminal, checkout relevant branch and run the gateway.

```bash
cd $path-to-kms-core
git checkout mano/update-to-latest-fhevm
cd blockchain/gateway
cargo run --bin gateway
```

Wait for the gateway to start listening for blocks and print block numbers.

```bash
...
2024-10-16T15:35:22.876765Z  INFO gateway::events::manager: 🧱 block number: 10
2024-10-16T15:35:27.787809Z  INFO gateway::events::manager: 🧱 block number: 11
2024-10-16T15:35:27.787809Z  INFO gateway::events::manager: 🧱 block number: 12
2024-10-16T15:35:27.787809Z  INFO gateway::events::manager: 🧱 block number: 13
...
```

5. From the fhevm repo, run one of the test for trivial decryption.

```bash
cd work_dir/fhevm; npx hardhat test --grep 'test async decrypt uint32'; cd ..
```

6. To tear down the setup, stop the docker containers:

```
make stop-coprocessor
make stop-full
```

The gateway will automatically exit as the connection will be closed from blockchain sideq

## Note on test results (for trivial encrypt)

1. PASSING TESTS

```bash
npx hardhat test --network localCoprocessor --grep 'test async decrypt bool$'
npx hardhat test --network localCoprocessor --grep 'test async decrypt uint4$'
npx hardhat test --network localCoprocessor --grep 'test async decrypt uint8$'
npx hardhat test --network localCoprocessor --grep 'test async decrypt uint16$'
npx hardhat test --network localCoprocessor --grep 'test async decrypt uint32$'
```


2. FAILING TESTS

```bash
npx hardhat test --network localCoprocessor --grep 'test async decrypt bool trustless$'
npx hardhat test --network localCoprocessor --grep 'test async decrypt uint64$'
npx hardhat test --network localCoprocessor --grep 'test async decrypt address$'
```

The tests for `uint64` and `address` types seem to fail because of
serialization error. These should are being investigated.

The reason for failure in bool trustless is yet to be explored.
