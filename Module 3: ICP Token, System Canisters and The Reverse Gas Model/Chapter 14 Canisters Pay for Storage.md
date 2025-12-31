# Canisters Pay for Storage

The Internet Computer Protocol charges canisters a recurring fee in `Cycles` for the on-chain storage space they occupy. The storage space a canister is accounted for includes both its **bytecode** and **state**.

### Storage Fees

As a general overview, the storage fee of **1 Gib** of data is **127\_000 Cycles** per second. Over a year, it amounts to 4 trillion `Cycles` or around $5. The fees are taken out of the Canisters Cycles Balance.

You can observe the storage fees in real time by deploying a canister and monitor its Cycles balance over time.

## Proof of Storage Fees

Deploy any canister and **do not call any of its functions**:

```rust
dfx new do_nothing --type rust --no-frontend
dfx deploy
```

Check the Cycles balance of the canister with the command below and compare it with the balance 1 minute **after**:

```solidity
dfx canister status 
```

You will see that `Cycles Balance` **reduces** because **t**he network continuously deducts `Cycles` from the canister to pay for storage fees.

`Before`:

**Balance**: 2\_553\_98**1\_994\_860** Cycles

![Screenshot 2025-09-26 at 15.25.53.png](../.gitbook/assets/7e4b6f3a-20dc-4078-a912-016bd4469872.png)

`1 minute After`:

**Balance**: 2\_553\_98**0\_664\_965** Cycles

![Screenshot 2025-09-26 at 15.26.36.png](../.gitbook/assets/25d63def-3c3c-495e-9427-7a889bfd4bca.png)

The `Cycles` balance decreased by approximately 1,330,000 `Cycles` in just around one minute. The network continuously charges canisters for storing data on-chain, regardless of on-chain activities.

## How Storage Fees Are Calculated

Storage fees are charged a fixed amount of `Cycles` per byte, proportional to the amount of storage space the canister uses.

The network charges 127\_000 `Cycles` per second for 1 **Gib** of on-chain storage. Multiplied by a year, that’s about **$5** for 1 **Gib** of data:

* 1 GiB of storage ≈ $5 USD per year

Say that the our canister uses up 10 mb of on-chain storage space (both bytecode and state combined), then the cost break down is:

$$
1 Mib = 124 Cycles/second
$$

over a year it is:

$$
10 Mib \ \approx \ 0.05_{\text{USD}} /year
$$

Storage fees on ICP are relatively affordable, but you should still design your canister to prevent unbounded storage. Otherwise, malicious actors could spam your canister with data to inflate your costs.

### Monitoring Storage Costs

The `canister status` command provides real-time information about storage fees. Look for the key metric: **`Idle cycles burned per day`**.

```bash
dfx canister status <canister-id>
```

![Screenshot 2025-10-30 at 05.52.00.png](../.gitbook/assets/Screenshot_2025-10-30_at_05.52.00.png)

This metric shows how many `Cycles` are consumed daily just for storage fees when the canister is idle (not processing any requests). The amount of Cycles burned grows as your canister s up more state.

## What Happens When A Canister Runs Out Of Cycles?

If a canister runs out of `Cycles`, **all of its data will be deleted** from the protocol—this includes includes its **storage** and **bytecode**. Therefore, we need to actively monitor the canister’s `Cycles` balance and plan ahead.

Take into consideration that the canister has sufficient `Cycles` to cover:

1. **Computation costs** for user function calls.
2. **Storage fees**.

### Cycles Balance Safety Mechanism: The Freezing Threshold

Canisters have a safety mechanism called the **freezing threshold**. When the `Cycles` balance becomes low and falls to a certain threshold amount, the freezing threshold mechanism **stops executing all function calls to the canister.**

The freezing threshold acts as a safety buffer that protects your canister from running out of `Cycles` unexpectedly. This gives the developers time during the “frozen” period, to:

* top-up the canister with more `Cycles`
* or leave it be. The canister’s `Cycles` balance would eventually run out because of storage fees and the canister’s data will be deleted permanently from the protocol.

You can see the freezing threshold in the **canister status**, which indicates how long much time the canister has when the freezing threshold is activated before it runs out of `Cycles`.

![Screenshot 2025-10-30 at 06.01.16.png](../.gitbook/assets/Screenshot_2025-10-30_at_06.01.16.png)

**2\_592\_000** seconds converts to **30** days. During these **30** days grace period, the Cycles balance will be continually depleted by the storage fees and you need top up the canister with more `Cycles` or all of its data will be deleted.

The next article discusses how to perform inter-canister calls.
