
# D2 | Origins, Destinations & Economy

## 1. Network: Origins and Destinations

Reactive Network connects different blockchains through event monitoring and callbacks.

- **Origin Chain:** The chain where events happen and event logs are read from.
- **Reactive Network:** Listens to events and executes Reactive Contract logic through `react()`.
- **Destination Chain:** The chain where Reactive Network delivers callback transactions and state changes occur.

A Reactive Contract can listen to multiple Origin Chains and send callbacks to one or more Destination Chains.

### Mainnet and Testnet Compatibility

Mainnets and testnets must not be mixed.

- Mainnet Origin → Mainnet Destination
- Testnet Origin → Testnet Destination

For example:

```text
Base Sepolia (Origin)
        |
        | Event Logs
        v
Reactive Lasna
        |
        | react()
        | Callback Transaction
        v
Ethereum Sepolia (Destination)
```

### Lasna Testnet Information

| Item | Value |
|---|---|
| Network | Reactive Lasna |
| Chain ID | 5318007 |
| RPC | https://lasna-rpc.rnk.dev/ |
| System Contract | 0x8888888888888888888888888888888888888888 |

Source:
https://dev.reactive.network/origins-and-destinations

Sections: Overview, Mainnet Chains, Testnet Chains.

---

## 2. Query Lasna System Contract Parameters

I used Foundry `cast` in GitHub Codespaces to read three parameters from the Reactive Lasna System Contract.

### Environment

```text
cast Version: 1.9.0
```

### 2.1 Gas Price Coefficient

Command:

```bash
cast call 0x8888888888888888888888888888888888888888 "_gasPriceCoeffPer1000()" --rpc-url https://lasna-rpc.rnk.dev/
```

Actual output:

```text
0x00000000000000000000000000000000000000000000000000000000000003e8
```

Decimal value:

```text
1000
```

### 2.2 Extra Gas

Command:

```bash
cast call 0x8888888888888888888888888888888888888888 "_extraGas()" --rpc-url https://lasna-rpc.rnk.dev/
```

Actual output:

```text
0x00000000000000000000000000000000000000000000000000000000000186a0
```

Decimal value:

```text
100000
```

### 2.3 Maximum Charge Gas

Command:

```bash
cast call 0x8888888888888888888888888888888888888888 "_maxChargeGas()" --rpc-url https://lasna-rpc.rnk.dev/
```

Actual output:

```text
0x000000000000000000000000000000000000000000000000000000000000c350
```

Decimal value:

```text
50000
```

### Query Results Summary

| Parameter | Decimal Value | Meaning |
|---|---:|---|
| `_gasPriceCoeffPer1000()` | 1000 | Gas pricing coefficient |
| `_extraGas()` | 100000 | Fixed gas surcharge |
| `_maxChargeGas()` | 50000 | Gas budget cap for payment settlement |

The queried values match the initial defaults documented by Reactive Network. The owner can change these parameters, so querying the System Contract is more reliable than assuming the defaults.

Source:
https://dev.reactive.network/economy

Section: Fee Model.

---

## 3. How Is One react() Execution Charged?

According to the official documentation, the fee formula is:

```text
fee = p_base × C × (g_used + K)
```

Where:

- `p_base`: The greater of `tx.gasprice` and `block.basefee`.
- `C`: Pricing coefficient, obtained from `_gasPriceCoeffPer1000()` divided by 1000.
- `g_used`: Actual gas consumed during execution.
- `K`: Fixed gas surcharge, obtained from `_extraGas()`.

Using my actual Lasna query results:

```text
C = 1000 / 1000 = 1

K = 100000
```

Therefore, the current fee formula based on these queried parameters is:

```text
fee = p_base × 1 × (g_used + 100000)
```

### Example (Illustrative Values)

Assume:

- Base gas price = 10 Gwei
- Actual gas used = 200,000
- Extra gas = 100,000
- Coefficient = 1

Then:

```text
fee = 10 × 1 × (200000 + 100000)
    = 3000000 Gwei
    = 0.003 ETH
```

This is an illustrative calculation, not an actual transaction fee.

### What does _maxChargeGas() do?

My query returned:

```text
_maxChargeGas() = 50000
```

This parameter does not directly enter the fee formula.

Instead, it caps the gas available for the `pay()` function when the System Contract attempts to settle outstanding payment.

Source:
https://dev.reactive.network/economy

Sections: Fee Model, Gas Limits, Funding Reactive Contracts.

---

## 4. Balance, Reserves and Debt

Three separate values determine whether a contract can continue executing.

| Concept | Meaning | Held by |
|---|---|---|
| Balance | Funds the contract itself can spend | Reactive Contract |
| Reserves | Prepaid credits used to pay execution fees | System Contract |
| Debt | Outstanding unpaid execution fees | System Contract |

### How do they interact?

1. Execution charges consume Reserves first.
2. When Reserves are insufficient, Debt accrues.
3. Deposits clear outstanding Debt first.
4. Any remaining deposited funds are added to Reserves.

Balance is separate from Reserves and Debt.

A direct transfer to the Reactive Contract increases its Balance, but does not automatically create Reserves or settle Debt.

### Why does any Debt stop execution?

The System Contract checks Debt before executing `react()`.

If:

```text
Debt > 0
```

The System Contract will not call `react()`.

This prevents contracts with unpaid execution fees from continuing to consume network resources.

To fund a contract and clear its Debt, `depositTo()` can be called on the System Contract. It settles outstanding Debt first, then adds the remaining funds to Reserves.

Source:
https://dev.reactive.network/economy

Sections: Balances, System Contract Interactions, Funding Reactive Contracts.

---

## 5. Is a Reverted react() Still Charged?

Yes.

A reverted `react()` is still charged.

The System Contract catches the revert, records a `ReactiveContractReverted` event, and continues to the charging step.

The fee is still calculated based on gas actually consumed. There is no refund simply because the execution reverted.

Therefore:

```text
Revert ≠ Free Execution
```

### Implications for Smart Contract Design

- Validate inputs before performing expensive operations.
- Avoid unnecessary reverts.
- Handle expected failure conditions carefully.
- Maintain sufficient Reserves.
- Monitor Debt to prevent service interruption.

A revert should not be treated as a free failure-handling mechanism.

Source:
https://dev.reactive.network/economy

Sections: System Contract Interactions, Fee Model.

---

## 6. Key Takeaways

1. Origin is where events happen; Destination is where callbacks are delivered.
2. Mainnet and testnet chains must not be mixed.
3. Reactive execution fees depend on gas price, pricing coefficient, actual gas consumption and fixed additional gas.
4. Reserves are prepaid credits; Debt represents unpaid fees; Balance is separately controlled by the contract.
5. Any outstanding Debt blocks further execution.
6. A reverted `react()` is still charged.
7. `_maxChargeGas()` limits the gas available for payment settlement, not the gas used directly in the fee formula.

## 7. Official References

1. Reactive Network - Origins & Destinations  
   https://dev.reactive.network/origins-and-destinations

2. Reactive Network - Economy & System Interactions  
   https://dev.reactive.network/economy
