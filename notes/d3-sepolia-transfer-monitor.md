# D3 | Sepolia ERC-20 Transfer Monitoring

## 1. Objective

Deploy a Reactive Smart Contract on Lasna to monitor ERC-20 Transfer events on Ethereum Sepolia.

The contract automatically reacts to matching transfer events, records transfer amounts, and unsubscribes after processing three events.

## 2. Architecture

```text
Ethereum Sepolia
    |
    | ERC-20 Transfer Event
    v
Reactive Network (Lasna)
    |
    | Event Detection
    v
XfersMonitor.sol
    |
    | react()
    v
Record Transfer Volume
    |
    | After 3 Events
    v
Automatic Unsubscription
```

## 3. Deployment Information

### Test Token

- Name: Camp Test Token
- Symbol: CTT
- Network: Ethereum Sepolia
- Chain ID: 11155111
- Token Address: `0xd70080970b5D9f0501c7457c46AE3BAEe5AEb909`

### Reactive Contract

- Network: Lasna
- Contract Address: `0xf42dBf3076d8129d8cdd4c9f3149953114ABfF18`
- [View on Reactscan](https://lasna.reactscan.net/address/0xf42dbf3076d8129d8cdd4c9f3149953114abff18)

## 4. Subscription Configuration

The contract subscribes to ERC-20 Transfer events emitted by the test token on Ethereum Sepolia.

Filtering conditions:

- Chain ID: 11155111
- Contract address: The deployed Camp Test Token address
- topic0: ERC-20 Transfer event signature
- Remaining topics: REACTIVE_IGNORE

The contract processes a maximum of three matching events.

After reaching the limit, it automatically calls `SYSTEM.unsubscribe()`.

## 5. Reactive Execution

A successful Reactive execution was verified on Reactscan.

- Transaction Hash: `0x01de861631561b931da3b4658e347d7277b3d99801033c38203cbc84274747bb`
- [View Transaction on Reactscan](https://lasna.reactscan.net/tx/0x01de861631561b931da3b4658e347d7277b3d99801033c38203cbc84274747bb)
- Status: SUCCESS
- Method: `trigger()`

The transaction confirms that the Reactive System triggered the monitoring contract with a Sepolia event.

## 6. Verification Results

| Metric | Result |
|---|---:|
| Maximum executions | 3 |
| Remaining executions | 0 |
| Total transfer volume | 3 CTT |
| Last Reactive block | 9211115 |
| Subscription after completion | 0 |

The contract successfully processed three matching Transfer events and reached its execution limit.

## 7. Key Learnings

1. Reactive contracts can subscribe to events on external blockchain networks.
2. `SYSTEM.subscribe()` establishes event subscriptions.
3. `react()` processes events delivered by the Reactive System.
4. `onlySystem` restricts execution to the Reactive System.
5. `REACTIVE_IGNORE` allows wildcard matching for selected event topics.
6. `SYSTEM.unsubscribe()` stops further event monitoring.
7. Contract balance, reserves and debt are important for maintaining execution and fee settlement.

## 8. Official References

- [Reactive Contracts Documentation](https://dev.reactive.network/reactive-contracts)
- [Xfers.sol Official Example](https://github.com/Reactive-Network/reactive-smart-contract-demos/blob/omni/src/micro-demos/Xfers.sol)
- [Reactive Economy](https://dev.reactive.network/economy)
- [Reactscan Lasna](https://lasna.reactscan.net/)

## 9. Conclusion

Successfully deployed a Reactive monitoring contract on Lasna that listens to ERC-20 Transfer events on Ethereum Sepolia.

The integration demonstrated cross-chain event monitoring, automated contract execution, execution-limit control, and automatic unsubscription.
