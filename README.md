# chainbridge-solidity

## Dependencies

Requires `nodejs` and `npm`.

## Commands

`make install-deps`: Installs truffle and ganache globally, fetches local dependencies. Also installs `abigen` from `go-ethereum`.

`make bindings`: Creates go bindings in `./build/bindings/go`

`PORT=<port> SILENT=<bool> make start-ganache`: Starts a ganache instance, default `PORT=8545 SILENT=false`

`QUIET=<bool> make start-geth`: Starts a geth instance with test keys

`PORT=<port> make deploy`: Deploys all contract instances, default `PORT=8545`

`make test`: Runs truffle tests.

`make compile`: Compile contracts.

## cb-sol-cli (JS CLI)

This is a small CLI application to deploy the contracts and interact with the chain. It consists of four main sub-commands `deploy`, `erc20`, `erc721`, and `cent`. To install run `make install-cli`.

#### Global Flags
```
  -p, --port <value>     Port of RPC instance (default: 8545)
  -h, --host <value>     Host of RPC instance (default: "127.0.0.1")
  --private-key <value>  Private key to use (default: "0x000000000000000000000000000000000000000000000000000000616c696365")
  -h, --help             display help for command
```
### `deploy`

Deploy contracts with configurable constructor arguments. Relayers will be added from default keys (max 5).


```
$ cb-sol-cli deploy

Options:
  --chain-id <value>           Chain ID for the instance (default: 0)
  --relayers <value>           List of initial relayers (default: ["0xff93B45308FD417dF303D6515aB04D9e89a750Ca","0x8e0a907331554AF72563Bd8D43051C2E64Be5d35","0x24962717f8fA5BA3b931bACaF9ac03924EB475a0","0x148FfB2074A9e59eD58142822b3eB3fcBffb0cd7","0x4CEEf6139f00F9F4535Ad19640Ff7A0137708485"])
  --relayer-threshold <value>  Number of votes required for a proposal to pass (default: 2)
```

### `erc20`

#### - `mint` 
Mint default erc20 tokens.

```
$ cb-sol-cli erc20 mint

Options:
  --value <amount>          Amount to mint (default: 100)
  --erc20Address <address>  Custom erc20 address (default: "0x3f709398808af36ADBA86ACC617FeB7F5B7B193E")
```

#### - `transfer`
Initiate a transfer of erc20 to some destination chain.
```
$ cb-sol-cli erc20 transfer

Options:
  --value <amount>                 Amount to transfer (default: 1)
  --dest <value>                   destination chain (default: 1)
  --recipient <address>            Destination recipient address (default: "0x4CEEf6139f00F9F4535Ad19640Ff7A0137708485")
  --erc20Address <address>         Custom erc20 address (default: "0x3f709398808af36ADBA86ACC617FeB7F5B7B193E")
  --erc20HandlerAddress <address>  Custom erc20Handler contract (default: "0x2B6Ab4b880A45a07d83Cf4d664Df4Ab85705Bc07")
  --bridgeAddress <address>        Custom bridge address (default: "0x3167776db165D8eA0f51790CA2bbf44Db5105ADF")
```

#### - `balance`
Check the balance of an account.
```
$ cb-sol-cli erc20 balance

Options:
  --address <address>       Address to query (default: "0xff93B45308FD417dF303D6515aB04D9e89a750Ca")
  --erc20Address <address>  Custom erc20 address (default: "0x3f709398808af36ADBA86ACC617FeB7F5B7B193E")
```

### `erc721`

#### - `mint`
Mint default erc721 tokens.

```
$ cb-sol-cli erc721 mint

Options:
  --erc721Address <address>  Custom erc721 contract (default: "0x21605f71845f372A9ed84253d2D024B7B10999f4")
  --id <id>                  ERC721 token id (default: 1)
```

#### - `transfer`
Initiate a transfer of erc721 to some destination chain.
```
$ cb-sol-cli erc721 transfer

  --id <id>                         ERC721 token id (default: 1)
  --dest <value>                    destination chain (default: 1)
  --recipient <address>             Destination recipient address (default: "0x4CEEf6139f00F9F4535Ad19640Ff7A0137708485")
  --erc721Address <address>         Custom erc721 contract
  --erc721HandlerAddress <address>  Custom erc721 handler
  --bridgeAddress <address>         Custom bridge address (default: "0x3167776db165D8eA0f51790CA2bbf44Db5105ADF")
```

### Centrifuge (`cent`)

#### - `transferHash`
Initiate transfer of a hash.

```
$ cb-sol-cli cent transferHash

Options:
  --hash <value>           The hash that will be transferred (default: "0x0000000000000000000000000000000000000000000000000000000000000001")
  --dest-id <value>        The cahin where the deposit will finalize (default: 1)
  --centAddress <value>    Centrifuge handler contract address (default: "0xc279648CE5cAa25B9bA753dAb0Dfef44A069BaF4")
  --bridgeAddress <value>  Bridge contract address (default: "0x3167776db165D8eA0f51790CA2bbf44Db5105ADF")
```

#### - `getHash`
Verify transfer of hash:

```
$ cb-sol-cli cent getHash

Options:
  --hash <value>         A hash to lookup (default: "0x0000000000000000000000000000000000000000000000000000000000000001")
  --centAddress <value>  Centrifuge handler contract address (default: "0xc279648CE5cAa25B9bA753dAb0Dfef44A069BaF4")
```

# ChainBridge-Solidity Data Layout

## ERC20Handler.sol

### deposit

```   
function deposit(
    uint256 destinationChainID,
    uint256 depositNonce,
    address depositer,
    bytes memory data
    ) public override _onlyBridge
```
`bytes memory data` is laid out as following:
```
originChainTokenAddress     address   - @0x20
amount                      uint256   - @0x40
destinationRecipientAddress           - @0x60 - END
```

### executeDeposit

```
function executeDeposit(bytes memory data) public override _onlyBridge
```
`bytes memory data` is laid out as following (since we know `len(tokenID) = 64`):

```
amount                      uint256   - @0x20 - 0x40
tokenID                               - @0x40 - 0xC0
-----------------------------------------------------
tokenID len                 uint256   - @0x40 - 0x60
tokenID                     bytes     - @0x60 - 0xA0
-----------------------------------------------------
destinationRecipientAddress           - @0xA0 - END
-----------------------------------------------------
destinationRecipientAddress len uint256 - @0xA0 - 0xC0
destinationRecipientAddress     bytes   - @0xC0 - END

```

### tokenID in ERC20Handler is different from other tokenIDs

`tokenID` is a `bytes` array laid out as follows:

```
chainID                     uint256   - @0x00 - 0x20
tokenAddress                address   - @0x20 - 0x40

```

## ERC721Handler.sol

### deposit

```
function deposit(
    uint256 destinationChainID, 
    uint256 depositNonce, 
    address depositer, 
    bytes memory data
    ) public override _onlyBridge
```

`bytes memory data` is laid out as following:
```
originChainTokenAddress        address   - @0x20 - 0x40
destinationChainTokenAddress   address   - @0x40 - 0x60
destinationRecipientAddress    address   - @0x80 - 0xA0
tokenID                        uint256   - @0xA0 - 0xC0
metaData                                 - @0xC0 - END
------------------------------------------------------
metaData length declaration    uint256   - @0xC0 - 0xE0
metaData                       bytes     - @0xE0 - END
```

### executeDeposit

```
function executeDeposit(bytes memory data) public override _onlyBridge
```

`bytes memory data` is laid out as following:
```
destinationChainTokenAddress   address   - @0x20 - 0x40
destinationRecipientAddress    address   - @0x40 - 0x60
tokenID                        uint256   - @0x60 - 0x80
metaData                                 - @0x80 - END
------------------------------------------------------
metaData length declaration    uint256   - @0x80 - 0xA0
metaData                       bytes     - @0xA0 - END
```

## GenericHandler.sol

### deposit

```
function deposit(
    uint256 destinationChainID, 
    uint256 depositNonce, 
    address depositer, 
    bytes memory data
    ) public override _onlyBridge
```

`bytes memory data` is laid out as following:
```

destinationRecipientAddress    address   - @0x20 - 0x40
metaData                                 - @0x40 - END
------------------------------------------------------
metaData length declaration    uint256   - @0x40 - 0x60
metaData                       bytes     - @0x60 - END
```

### executeDeposit

Currently unimplemented.




