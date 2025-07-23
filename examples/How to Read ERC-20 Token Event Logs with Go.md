# How to Read ERC-20 Token Event Logs with Go  

## Setting Up the ERC-20 Contract Interface  

To interact with Ethereum-based tokens, we begin by creating a Solidity interface for the ERC-20 standard. This interface defines the events we'll monitor:  

```solidity
// erc20.sol
pragma solidity ^0.4.24;
contract ERC20 {
 event Transfer(address indexed from, address indexed to, uint tokens);
 event Approval(address indexed tokenOwner, address indexed spender, uint tokens);
}
```  

👉 [Explore Ethereum development tools on OKX](https://bit.ly/okx-bonus)  

## Generating Go Bindings from Solidity ABI  

Using `abigen`, we convert the Solidity contract's ABI into Go code for seamless integration:  

```bash
solc --abi erc20.sol
abigen --abi=erc20_sol_ERC20.abi --pkg=token --out=erc20.go
```  

This generates type-safe Go structures representing the contract's events and methods.  

## Defining Event Structures in Go  

Create Go structs that mirror the event signatures:  

```go
type LogTransfer struct {
 From       common.Address
 To         common.Address
 Tokens     *big.Int
}

type LogApproval struct {
 TokenOwner common.Address
 Spender    common.Address
 Tokens     *big.Int
}
```  

## Connecting to an Ethereum Node  

Establish a connection to a blockchain explorer API endpoint:  

```go
client, err := ethclient.Dial("https://cloudflare-eth.com")
if err != nil {
 log.Fatal(err)
}
```  

👉 [Discover secure Ethereum APIs on OKX](https://bit.ly/okx-bonus)  

## Creating a Filter Query  

Specify the contract address and block range for event monitoring:  

```go
contractAddress := common.HexToAddress("0xe41d2489571d322189246dafa5ebde1f4699f498")
query := ethereum.FilterQuery{
 FromBlock: big.NewInt(6383820),
 ToBlock:   big.NewInt(6383840),
 Addresses: []common.Address{contractAddress},
}
```  

## Fetching and Processing Event Logs  

Retrieve logs using the Ethereum client:  

```go
logs, err := client.FilterLogs(context.Background(), query)
if err != nil {
 log.Fatal(err)
}
```  

Parse the contract ABI for event decoding:  

```go
contractAbi, err := abi.JSON(strings.NewReader(string(token.TokenABI)))
if err != nil {
 log.Fatal(err)
}
```  

## Calculating Event Signatures  

Generate Keccak256 hashes for event identifiers:  

```go
logTransferSig := []byte("Transfer(address,address,uint256)")
logApprovalSig := []byte("Approval(address,address,uint256)")
logTransferSigHash := crypto.Keccak256Hash(logTransferSig)
logApprovalSigHash := crypto.Keccak256Hash(logApprovalSig)
```  

## Parsing Event Data  

Iterate through logs and decode relevant events:  

```go
for _, vLog := range logs {
 fmt.Printf("Block Number: %d\n", vLog.BlockNumber)
 switch vLog.Topics[0].Hex() {
 case logTransferSigHash.Hex():
  var transfer LogTransfer
  contractAbi.Unpack(&transfer, "Transfer", vLog.Data)
  transfer.From = common.HexToAddress(vLog.Topics[1].Hex())
  transfer.To = common.HexToAddress(vLog.Topics[2].Hex())
  fmt.Printf("Transfer: %s → %s | Amount: %s\n", transfer.From, transfer.To, transfer.Tokens.String())
 case logApprovalSigHash.Hex():
  var approval LogApproval
  contractAbi.Unpack(&approval, "Approval", vLog.Data)
  approval.TokenOwner = common.HexToAddress(vLog.Topics[1].Hex())
  approval.Spender = common.HexToAddress(vLog.Topics[2].Hex())
  fmt.Printf("Approval: %s authorized %s | Amount: %s\n", approval.TokenOwner, approval.Spender, approval.Tokens.String())
 }
}
```  

👉 [Learn advanced blockchain analysis on OKX](https://bit.ly/okx-bonus)  

---

## FAQ: ERC-20 Event Log Processing  

### What are ERC-20 event logs?  
Event logs are immutable records of contract activities on the Ethereum blockchain. The `Transfer` and `Approval` events specifically track token movements and spending permissions.  

### Why use Go for Ethereum log analysis?  
Go offers strong concurrency support, efficient memory management, and robust libraries like `go-ethereum` for blockchain interactions.  

### How do indexed event parameters work?  
Indexed parameters are stored in `topics` rather than log data, enabling efficient filtering through bloom filters.  

### What is the purpose of the FilterQuery?  
Filter queries allow developers to target specific contracts, timeframes, and event types for efficient data retrieval.  

### How does event signature hashing work?  
The Keccak256 hash of the event signature (e.g., `Transfer(address,address,uint256)`) creates a unique identifier for event filtering.  

### What tools simplify Ethereum development?  
Frameworks like `abigen` automate Go binding generation, while clients like `ethclient` handle RPC communications with Ethereum nodes.  

---

## Complete Code Example  

```go
// event_read_erc20.go
package main

import (
 "context"
 "fmt"
 "log"
 "math/big"
 "strings"
 "github.com/ethereum/go-ethereum"
 "github.com/ethereum/go-ethereum/accounts/abi"
 "github.com/ethereum/go-ethereum/common"
 "github.com/ethereum/go-ethereum/crypto"
 "github.com/ethereum/go-ethereum/ethclient"
 token "./contracts_erc20" // Generated binding
)

func main() {
 client, _ := ethclient.Dial("https://cloudflare-eth.com")
 contractAddress := common.HexToAddress("0xe41d2489571d322189246dafa5ebde1f4699f498")

 query := ethereum.FilterQuery{
  FromBlock: big.NewInt(6383820),
  ToBlock:   big.NewInt(6383840),
  Addresses: []common.Address{contractAddress},
 }

 logs, _ := client.FilterLogs(context.Background(), query)
 contractAbi, _ := abi.JSON(strings.NewReader(string(token.TokenABI)))

 logTransferSigHash := crypto.Keccak256Hash([]byte("Transfer(address,address,uint256)"))
 logApprovalSigHash := crypto.Keccak256Hash([]byte("Approval(address,address,uint256)"))

 for _, vLog := range logs {
  switch vLog.Topics[0].Hex() {
  case logTransferSigHash.Hex():
   var transfer struct {
    From   common.Address
    To     common.Address
    Amount *big.Int
   }
   contractAbi.Unpack(&transfer, "Transfer", vLog.Data)
   transfer.From = common.HexToAddress(vLog.Topics[1].Hex())
   transfer.To = common.HexToAddress(vLog.Topics[2].Hex())
   fmt.Printf("Transfer: %s → %s | %s\n", transfer.From.Hex(), transfer.To.Hex(), transfer.Amount.String())

  case logApprovalSigHash.Hex():
   var approval struct {
    Owner  common.Address
    Spender common.Address
    Amount *big.Int
   }
   contractAbi.Unpack(&approval, "Approval", vLog.Data)
   approval.Owner = common.HexToAddress(vLog.Topics[1].Hex())
   approval.Spender = common.HexToAddress(vLog.Topics[2].Hex())
   fmt.Printf("Approval: %s → %s | %s\n", approval.Owner.Hex(), approval.Spender.Hex(), approval.Amount.String())
  }
 }
}
```  

This implementation demonstrates:  
- Ethereum client setup  
- Event signature hashing  
- Log filtering mechanics  
- Structured event parsing  
