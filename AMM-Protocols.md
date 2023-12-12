# Understanding the Vulnerabilities of AMM Protocols
# Intro

In the ever-evolving landscape of decentralized finance ([DeFi](https://www.coinbase.com/ru/learn/crypto-basics/what-is-defi)), the quest for an efficient exchange that accommodates both makers and takers within smart contracts has been relentless. Traditional financial markets rely on the order book model, where makers set prices and amounts, and takers execute transactions accordingly. However, the blockchain’s unique characteristics have presented challenges in adopting this model, particularly due to the impracticality of handling frequent order placements and cancellations within the constraints of lengthy block generation times.

[Automated Market Makers (AMM)](https://academy.shrimpy.io/lesson/what-is-an-amm-automated-market-maker) simplified the maker’s actions. By allowing makers to supply multiple tokens to a single liquidity pool, AMMs automatically calculate the current value and exchange rate based on the pool’s balance. Takers can then seamlessly execute transactions without direct interaction with makers. While this innovation streamlines the process, it introduces its own set of challenges.

This article explores key features, protocol types and the core security vulnerabilities in AMM protocols, shedding light on the drawbacks that auditors and developers must navigate.

# Key Features

### Token Pair Pools
As a user, you can create liquidity pools by combining two different tokens. For instance, you can pair a stablecoin like USDC with another token, such as ETH, thereby forming a trading pair like USDC/ETH.

### User can provide liquidity
You contribute $100,000 to the USDC/ETH token pair, allocating $50,000 to USDC and $50,000 to ETH. Upon completion of this transaction, 100,000 liquidity provider tokens (LPT) are automatically credited to your address, representing your contribution to the liquidity pool.

### Trade opportunities with assets
Selecting a trading pair, like USDC and ETH, involves specifying the amount of USDC tokens for selling or buying in exchange for ETH through the AMM platform interface. The AMM algorithm automatically calculates the price based on changes in the liquidity pool after the transaction, updating the pool with a new token distribution upon completion of the operation.

### Flash swaps
Instant swap feature that allows you, as a user, to instantly borrow tokens from liquidity pools, with the condition to return the borrowed amount plus fees or provide an equivalent amount of another token by the end of the transaction

### Price oracles
An oracle that provides asset price information to a blockchain.

# Protocol Types

The audit process of an AMM protocol largely depends on its type.
### 1. Constant product without concentrated liquidity

#### **Uniswap V1/V2**

- Uniswap V2 notably improved liquidity pools by enabling flexible ERC-20/ERC-20 token pairs, eliminating the need for ETH as the primary currency, in comparison with Uniswap V1.
    
- Uniswap V2 enhanced the price oracle system using existing liquidity pools for more accurate and tamper-resistant pricing. It also introduced flash swaps, allowing users to instantly borrow tokens.
    
- Pairs act as automated liquidity providers, standing ready to accept one token for the other as long as the “constant product” formula is preserved. This formula, most simply expressed as _x * y = k_, states that trades must not change the _product (k)_ of a pair’s reserve balances _(x and y)_.

#### **Balancer**

- Liquid pools on Balancer can consist of any proportion of up to 8 cryptocurrencies. This means that each token in the Balancer pool can have a different weight or percentage of the total pool value, offering more complex strategies for liquidity providers.
    
- The protocol features two pool types: public and private. Public pools are open to all, with unchangeable parameters post-creation. Private pools offer creators greater control, enabling management of tokens, weights, fees, and transaction pausing.
    
- The formula for a constant-product pool of assets is an extension of Uniswap’s **_x * y = k_**. For three assets, it would look like so: **_x * y * z = k_**, where: **_x_**, **_y_**, z  are balances of each token in a pool.
### **2. Stable swap pools**

#### **Curve**

- The main swap for behaviorally similar assets, such as stablecoins, or wrapped versions of similar assets, such as wBTC and tBTC.
    
- Implements the creation of ternary pools, leading to a reduction in costs associated with fees and slippage during operations.
    
- Applies the formula _x*y*z = k_, where _x_, _y_, and _z_ represent token balances in the pool, to ensure the constancy of the product of _balances (k)_.
### **3. Constant product with concentrated liquidity**

#### **Uniswap V3**

- Concentrated liquidity is the main concept behind V3. Instead of evenly distributing liquidity across the entire price curve, liquidity providers can optimize capital efficiency by focusing on a specific price range, like $1,800 to $1,900 per ETH, instead of spreading liquidity evenly.
    
- This targeted approach increases the likelihood of participating in trades within a high-volume area, maximizing potential trading revenue.
    
- The amount of liquidity provided can be measured by the value _𝐿_, which is equal to the _√k_.

# **Common attack vectors**

The general concept of all attacks lies in manipulating the behavior of smart contracts to extract assets from them.

### **Reentrancy**

In a reentrancy attack, a function can be externally invoked during its execution, allowing it to be executed multiple times within a single transaction. This typically occurs when a contract calls another contract before completing the processing of its current state.

For a clearer understanding of this attack, let’s examine an example of the **Beanstalk’s Wells protocol**.

An attack on the Beanstalk protocol could have been carried out by exploiting a vulnerability in the removeLiquidity function. The primary vulnerability lies in the violation of the [CEI pattern](https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html).

During the attack, malicious actors could have utilized an external contract with an ERC-777 callback function and subsequently called `getReserves()` during the execution of the removeLiquidity function. This allows obtaining incorrect reserve values.

Since the update of reserves occurs in the `_setReserves function` after interacting with the external contract (via `safeTransfer`) rather than before it, attackers could manipulate the protocol’s state, creating potential critical vulnerabilities.

Although the removeLiquidity function is protected by the `nonReentrant` modifier, which prevents reentrant calls, the vulnerability still exists because of the read-only reentrancy. External contracts could invoke `getReserves()` and interfere with the process, especially if they utilize [ERC-777](https://docs.openzeppelin.com/contracts/3.x/erc777) tokens with a callback function.

**Source:** [https://github.com/solodit/solodit_content/blob/main/reports/Cyfrin/2023-06-16-Beanstalk wells.md#read-only-reentrancy](https://github.com/solodit/solodit_content/blob/main/reports/Cyfrin/2023-06-16-Beanstalk%20wells.md#read-only-reentrancy)

**More examples:**

- Balancer [https://forum.balancer.fi/t/reentrancy-vulnerability-scope-expanded/4345](https://forum.balancer.fi/t/reentrancy-vulnerability-scope-expanded/4345)
    
- Uniswap [https://blog.openzeppelin.com/exploiting-uniswap-from-reentrancy-to-actual-profit](https://blog.openzeppelin.com/exploiting-uniswap-from-reentrancy-to-actual-profit)
    
- Orion protocol [https://t.me/kotyaSec/23](https://t.me/kotyaSec/23)
    

### **Logic Flow**

This type of vulnerability signifies a misunderstanding of the developer of project documentation or incorrect execution of the smart contract logic, which can lead to undesirable outcomes or exploitation.

Let’s examine this vulnerability using the example of **Kyberswap**.

Logic Flow is associated with the incorrect handling of ticks in [CLMM](https://blog.hubbleprotocol.io/what-is-clmm/), which may result in double liquidity addition.

Users provide liquidity in tick-divided price ranges. Exploiting a state where `currentTick` sits on a tick range boundary, `nearestCurrentTick` miscalculated as `currentTick — 1` results in mining liquidity in the range (`currentTick`, `currentTick + n`). During a subsequent one-to-zero swap, this miscalculation causes a re-addition of the created liquidity.

The issue arises from, pre-mining, crossing the tick boundary adds liquidity l0. Mining adds l1 liquidity but also contributes to the tick range, leading to l0 + l1 liquidity addition upon crossing the tick boundary. In the end, l1 + l0 + l1 liquidity is added due to mining and crossing as two ticks become identical.

Hacker starts with 1000 ETH, 2,000,000 USDC pool. Using a flash loan swaps 5000 ETH for USDC at $1 (tick 0). Hacker calculates liquidity needed in (0, n) to deplete the pool of 6000 ETH. Half of this liquidity is minted, exploiting the vulnerability. The hacker swaps 6000 USDC for 6000 ETH, repays the flash loan, and keeps around 1000 ETH as profit, along with the received USDC.

**Source:** [https://100proof.org/kyberswap-post-mortem.html](https://100proof.org/kyberswap-post-mortem.html)

**More examples:**

- Sushi Trident [https://code4rena.com/reports/2021-09-sushitrident#h-01-flash-swap-call-back-prior-to-transferring-tokens-in-indexpool](https://code4rena.com/reports/2021-09-sushitrident#h-01-flash-swap-call-back-prior-to-transferring-tokens-in-indexpool) [https://code4rena.com/reports/2021-09-sushitrident-2#h-10-concentratedliquiditypoolburn-wrong-implementation](https://code4rena.com/reports/2021-09-sushitrident-2#h-10-concentratedliquiditypoolburn-wrong-implementation)
    
- Elastic Swap [https://quillaudits.medium.com/decoding-elastic-swaps-850k-exploit-quillaudits-9ceb7fcd8d1a](https://quillaudits.medium.com/decoding-elastic-swaps-850k-exploit-quillaudits-9ceb7fcd8d1a)
    
- 0x0 Privacy DEX [https://0x0ai.notion.site/0x0ai/0x0-Privacy-DEX-Exploit-25373263928b4f18b31c438b2a040e33](https://0x0ai.notion.site/0x0ai/0x0-Privacy-DEX-Exploit-25373263928b4f18b31c438b2a040e33)

### **Overflow/Underflow**

**Overflow:** Result exceeds the max value for a data type (e.g., uint8 wraps to 0 after 255).

**Underflow:** Result wraps to max value when decreasing below min value for a signed type (e.g., int8 wraps to 127 after -128).

A good example is the **Velodrome Protocol**.

This protocol employs a stable [curve formula](https://alvarofeito.com/articles/curve/) (**_x³y+y³x >= k_**) for stablecoin pairs, where a critical vulnerability exists due to rounding errors in the calculation of the **_invariant (k)_**.

Specifically, the `_k function` encounters a rounding error in the calculation of `variable _a`, leading to its nullification when **_x * y < 1e18_**. If the value of **_x * y_** is less than or equal to 1e18, a rounding error occurs. This error subsequently results in an incorrect and successful validation of the product constants within the `swap()` function, triggering unauthorized draining of the pool.

The attack scenario involves the first liquidity provider [minting](https://github.com/velodrome-finance/contracts/blob/80dd33824a51fd2b2162311ac3c44512e151bcff/contracts/Pair.sol#L344-L363) a small amount of liquidity to the pool, exploiting the rounding error to set the invariant k to zero. Subsequently, the attacker can repeatedly steal from the pool by minting and draining until the overflow of the total supply.

**Source**: [https://github.com/spearbit/portfolio/blob/master/pdfs/Velodrome-Spearbit-Security-Review.pdf](https://github.com/spearbit/portfolio/blob/master/pdfs/Velodrome-Spearbit-Security-Review.pdf)

It’s also worth considering **Rounding Error**, In the context of **Balancer**.

This problem manifests itself during token exchange operations in [Linear Pools](https://docs.balancer.fi/concepts/pools/linear.html) due to difficulties with rounding.

The error arises from the assumption that scaling operations can always be carried out with rounding down without serious consequences. In Linear Pools, exchanges occur with extremely small token amounts, leading to a gradual accumulation of rounding errors. This, in turn, can impact the token exchange rate in the pool.

The vulnerability involved the use of flash swaps, where borrowed tokens were exchanged for base and wrapped tokens and then returned at a lower rate. This provided the attacker with an opportunity to profit from rounding errors.

**Source**: [https://medium.com/balancer-protocol/rate-manipulation-in-balancer-boosted-pools-technical-postmortem-53db4b642492](https://medium.com/balancer-protocol/rate-manipulation-in-balancer-boosted-pools-technical-postmortem-53db4b642492)

### **Data Validation**

The absence or insufficient thoroughness in the validation and cleansing of input data provided by users.

**FraxSwap**’s use of rebasing tokens introduces a critical data validation vulnerability during long-term swaps.

Users can cancel these swaps at any time through the `cancelLongTermSwap()` function, reclaiming unsold tokens. However, when rebasing tokens are involved, discrepancies between the actual contract balance and `internal reserves[]` accounting may occur during the swap. This misalignment poses a risk of excessive token transfers to users during cancel and withdraw operations, depleting the contract’s balance prematurely and leading to transaction reversions.

Proper construction of rebasing tokens, as outlined in [Uniswap documentation](https://docs.uniswap.org/contracts/v2/reference/smart-contracts/common-errors), is essential to prevent user token loss.

**Source:** [https://github.com/trailofbits/publications/blob/master/reviews/FraxQ22022.pdf](https://github.com/trailofbits/publications/blob/master/reviews/FraxQ22022.pdf)

**More examples:**

- LIDO [https://github.com/oxor-io/public_audits/blob/master/Lido/Lido_v2_on_chain.pdf](https://github.com/oxor-io/public_audits/blob/master/Lido/Lido_v2_on_chain.pdf)
    
- SushiSwap [https://twitter.com/SlowMist_Team/status/1644936375924584449?s=20](https://twitter.com/SlowMist_Team/status/1644936375924584449?s=20)
    
- Biswap [https://twitter.com/MetaTrustAlert/status/1674814217122349056?s=20](https://twitter.com/MetaTrustAlert/status/1674814217122349056?s=20)

### **Incorrect integration**

Mistakes in integrating components or functions within a smart contract based on ready-made protocols or third-party libraries.

Let’s take a look at an **Uranium Finance** attack example.

In the [original Uniswap V2 code](https://github.com/Uniswap/v2-core/blob/ee547b17853e71ed4e0101ccfd52e70d5acded58/contracts/UniswapV2Pair.sol#L158-L187), the algorithm relies on a multiplier of 1000 for balance adjustments, ensuring the integrity of the **_K=XY_** invariant. However, during the fork to Uranium Finance, developers erroneously increased this multiplier to 10000 in two instances within the `swap()` function but failed to update the corresponding check at the end of the function.

In the Uranium Finance code, the multiplier was increased to 10000 in the same locations:

`uint balance0Adjusted = balance0.mul(10000).sub(amount0In.mul(16));`

`uint balance1Adjusted = balance1.mul(10000).sub(amount1In.mul(16));`

However, the critical oversight lies in the failure to update the check at the end, where the original 1000 multiplier is retained:

`require(balance0Adjusted.mul(balance1Adjusted) >= uint(_reserve0).mul(_reserve1).mul(1000**2), 'UraniumSwap: K');`

This inconsistency allowed a malicious actor to exploit the flaw, permitting the exchange of a negligible amount of one token for an unproportionally large quantity of another.

**Source:** [https://uraniumfinance.medium.com/exploit-d3a88921531c](https://uraniumfinance.medium.com/exploit-d3a88921531c)

**More examples:**

- BurgerSwap [https://twitter.com/Mudit__Gupta/status/1398156036574306304?s=20](https://twitter.com/Mudit__Gupta/status/1398156036574306304?s=20)
    
- SwaposV2Pair [https://twitter.com/BeosinAlert/status/1647552192243728385?s=20](https://twitter.com/BeosinAlert/status/1647552192243728385?s=20)
    
- NowSwap Protocol [https://twitter.com/BlockSecTeam/status/1438100688215560192?s=20](https://twitter.com/BlockSecTeam/status/1438100688215560192?s=20)
    

### **Price Manipulation**

Artificially influencing the price of an asset in the market to gain an advantage.

One common method of price manipulation involves using [flash loans](https://www.halborn.com/blog/post/what-is-a-flash-loan-attack) — collateral-free loans provided on exchanges within a single transaction. Malicious actors can utilize flash loans to instantly borrow a significant amount of funds and manipulate market prices, creating an artificial change in the asset’s price.

Another method involves impacting liquidity on DEXs. A perpetrator may execute a series of trading operations aimed at artificially increasing or decreasing trading volume, leading to a change in the asset’s price.

This manipulation occurred on the **PancakeSwap BH/USDT trading pair**.

In this case, the attacker targeted the [BH/USDT trading pair on PancakeSwap](https://twitter.com/BeosinAlert/status/1712139760813375973?s=20). By exchanging USDT for BH at a low price, the attacker managed to extract liquidity from the specified trading pair at a significantly inflated price. The attack concluded when the attacker repaid the flash loan and retained the profit.

For executing the attack on the BNB Chain, the attacker paid approximately $4.16 in fees. However, after orchestrating the price manipulation and leveraging the created slippage, the attacker succeeded in withdrawing approximately $1.27 million in USDT.

**Source**: [https://www.halborn.com/blog/post/explained-the-bh-token-hack-october-2023](https://www.halborn.com/blog/post/explained-the-bh-token-hack-october-2023)

**More examples:**

- Strips Finance [https://medium.com/amber-group/strips-finances-price-manipulation-vulnerability-explained-f912734a8a2](https://medium.com/amber-group/strips-finances-price-manipulation-vulnerability-explained-f912734a8a2)
    
- Beluga Protocol [https://twitter.com/AnciliaInc/status/1712666421476667547?s=20](https://twitter.com/AnciliaInc/status/1712666421476667547?s=20)
    
- swapX [https://twitter.com/BlockSecTeam/status/1630111965942018049?s=20](https://twitter.com/BlockSecTeam/status/1630111965942018049?s=20)
    

### **Access control**

The Access Control vulnerability is associated with the insufficient or incorrect implementation of mechanisms that control who has access to specific functions or data within a smart contract. This can result in unauthorized function calls.

Let’s explore this type of vulnerability using the example of **CEXISWAP**.

The contract was exploited through an unprotected `initialize() function`, allowing the attacker to establish themselves as the owner of the contract.

To execute the attack, the [Exploiter contract](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/cd93944c3ec3cc7f5f4170a5857b31d1c560eda0/src/test/CEXISWAP_exp.sol#L56) was created, and its `exploit()` function was invoked. Within `exploit()`, the `initialize()` function was triggered, setting the attacker as the owner of the CexiSwap contract. Subsequently, `upgradeToAndCall()` was called, providing the attacker’s address and invoking the `exploit2()` function.

In the `exploit2()` function, activated through `upgradeToAndCall()`, `delegatecall` was used to invoke the transfer function, facilitating the transfer of all USDT to the attacker’s account.

By assuming ownership of the contract and utilizing the `upgradeAndCall()` function, the attacker successfully moved USDT out of the vulnerable contract.

**Source:** [https://twitter.com/DecurityHQ/status/1704759560614126030?s=20](https://twitter.com/DecurityHQ/status/1704759560614126030?s=20)

**More examples**:

- LeetSwap [https://twitter.com/peckshield/status/1686209024587710464?s=20](https://twitter.com/peckshield/status/1686209024587710464?s=20)
    

### **Compiler bugs**

The vulnerability is related to errors in the programming language compiler itself, which can lead to unforeseen or incorrect results during the compilation process of smart contracts.

Such issues can result in incorrect processing of the source code, improper optimization, inaccurate gas cost estimation, and may impact the language version compatibility.

A good example is the **pETH/ETH Pool** exploit.

The vulnerability resulted from an error in the smart contract implementation and the use of outdated versions of the Vyper compiler with flaws in [recursion protection](https://cryptoslate.com/ordinals-2-0-reduces-bloat-brings-smart-contract-like-functionality-to-inscriptions/#:~:text=Smart%20Contract%20Functionality%20via%20recursion&text=This%20recursive%20functionality%20brings%20several,applications%20on%20the%20Bitcoin%20blockchain.).

The attacker utilized an 80,000 WETH flash loan through Balancer, exchanged them for ETH, and provided 40,000 ETH to the Curve [pETH/ETH pool](https://etherscan.io/token/0x9848482da3ee3076165ce6497eda906e66bb85c5), receiving 32,431.41 LP tokens. By burning these tokens, they withdrew 3,740 pETH and 34,316 ETH.

They then deposited another 40,000 ETH, creating an additional 82,182 LP tokens. Extracting 1,184.73 pETH and 47,506.53 ETH, while burning 10,272.84 Curve LP tokens. They exchanged 4,924 pETH for 4,285 ETH within the Curve pool, followed by wrapping 86,106.65 ETH into WETH. Finally, they returned 80,000 WETH to Balancer to repay the flash loan. The attack resulted in a profit of 6,106.65 WETH (approximately $11 million).

The exploit occurred due to a recursion error between the removal and addition of liquidity, allowing the attacker to infinitely increase pool tokens and make a fraudulent claim on the entire pool’s holdings.

**Source**: [https://hackmd.io/@LlamaRisk/BJzSKHNjn](https://hackmd.io/@LlamaRisk/BJzSKHNjn)

The best way to remove risks of compiler bugs in the protocol is to use **fuzzing** testing. Fuzzing testing helps also for eliminating risks related with business logic, as an example [after finding the vulnerability](https://100proof.org/kyberswap-post-mortem.html) in the Kyberswap protocol the fixes was implemented and all the risks were mitigated, however, due to the lack of fuzzing testing there was missed a case when the vulnerability [can still be exploited](https://github.com/paco0x/kyber-exploit-example#readme) by brute forcing the pool and finding special conditions, which lead to > 54 M $ loss.

### **SafeERC20**

Lack of return value checked in `transfer` and `transferFrom` functions.

For example, **Spartan Protocol**.

In the standard implementation of ERC20, when the balance is insufficient, functions like `transfer()` and `transferFrom()` return “false” instead of an error. This could lead to the creation of tokens without receiving sufficient funds.

To address this issue, it is recommended to use [SafeERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol) from OpenZeppelin when interacting with ERC20 tokens. This tool enhances the security of ERC20 function calls by checking the boolean return values and reverting the transaction in case of failure.

Additionally, SafeERC20 provides auxiliary methods for increasing or decreasing allowances, helping to mitigate potential front-running attacks.

**Source**: [https://code4rena.com/reports/2021-07-spartan#h-03-result-of-transfer--transferfrom-not-checked](https://code4rena.com/reports/2021-07-spartan#h-03-result-of-transfer--transferfrom-not-checked)

### **Centralization risks**

Due to the overpowered roles it’s possible to withdraw all tokens from the protocol, in case of private key leaks. For example lot’s of projects were hacked due to the vulnerability of the Profanity address generator in the [Vanity wallet](https://vanity-eth.tk/).

The attack on **Wintermute** was caused by a flaw in the Profanity algorithm, allowing the perpetrator to directly target compromised private keys of Wintermute users.

For maximum security in cryptography, a random value is typically chosen as the seed for a [cryptographic pseudorandom number generator (CPRNG)](https://fuchsia.dev/fuchsia-src/concepts/kernel/cprng) to create private keys. However, Profanity used a 32-bit number as the seed for its CPRNG, enabling attackers to systematically guess values and reconstruct private keys.

In the case of Wintermute, this impacted both their DeFi vault contract and hot wallet, which were vanity addresses.

**Source**: [https://www.halborn.com/blog/post/explained-the-wintermute-hack-september-2022](https://www.halborn.com/blog/post/explained-the-wintermute-hack-september-2022)

### **Deflationary Token**

Deflationary tokens are a type of digital currency or token designed to reduce their overall supply over time. This reduction is commonly implemented by burning, which involves permanently removing a small percentage of each transaction from circulation.

The aim is to enhance the value of the remaining tokens in circulation by creating scarcity, leading to increased demand.

Two complex transactions were sent to the Ethereum Mainnet, leading to an attack on two **Balancer pools**.

The attacker used a smart contract to automate actions. Initially, a FlashLoan of 104,000 WETH was borrowed from the dYdX platform, following which WETH was exchanged for STA(STATERA) 24 times back and forth. This led to draining the STA balance from the pool, reducing it to 1 weiSTA due to the deflationary model of STA. The Balancer pool received 1% less STA in each exchange due to the applied fee.

Next, the hacker exchanged 1 weiSTA for WETH several times, causing STA to exit the pool while leaving WETH untouched. Similarly, balances of WBTC, SNX, and LINK tokens were depleted from the pool.

In the final stage, the attacker repaid the FlashLoan of 104,000 WETH on the dYdX platform. The attacker increased their share in the Balancer pool by depositing a small amount of weiSTA. Then, the collected tokens from the pool were exchanged for 136,000 STA through Uniswap V2, and subsequently, 136,000 STA were exchanged for 109 WETH.

**Source**: [https://blog.1inch.io/balancer-hack-2020/](https://blog.1inch.io/balancer-hack-2020/)

**More examples:**

- AES [https://twitter.com/BeosinAlert/status/1600478366255427584?s=20](https://twitter.com/BeosinAlert/status/1600478366255427584?s=20)
    
- TATA [https://twitter.com/BeosinAlert/status/1628636827279331330?s=20](https://twitter.com/BeosinAlert/status/1628636827279331330?s=20)

### **Slippage**

AMM protocols are susceptible to changes in available token liquidity, especially during periods of high volatility. This means that the expected transaction price may differ from the actual executed price.

The vulnerability in **Derby Finance** arises from a lack of slippage protection during swap transactions, specifically within the `Vault.claimTokens()` and `MainVault.withdrawRewards()` functions.

[The Swaps library](https://github.com/sherlock-audit/2023-01-derby/blob/0b520498fc14be28d158ec7dcc5d1d832570b1d1/derby-yield-optimiser/contracts/libraries/Swap.sol#L60-L97) calculates slippage parameters within the swap transaction, resulting in an inaccurate assessment of the minimum output value. This exposes the protocol to potential [front-running attacks](https://www.halborn.com/blog/post/what-is-a-front-running-attack), including [sandwich attacks](https://coinmarketcap.com/academy/article/what-are-sandwich-attacks-in-defi-and-how-can-you-avoid-them), where malicious actors exploit the absence of slippage protection.

Users can trigger the `withdrawRewards()` function to receive rewards from the protocol, and during this process, a portion of the rewards is exchanged for DERBY tokens through the Uniswap pool and transferred to the user.

The vulnerability jeopardizes the security of this exchange process, emphasizing the need for external slippage calculations and reliance on reputable oracles for accurate market prices to mitigate the risk effectively.

**Source**: [https://github.com/sherlock-audit/2023-01-derby-judging/issues/64](https://github.com/sherlock-audit/2023-01-derby-judging/issues/64)

**More examples**:

- Uniswap [https://twitter.com/bertcmiller/status/1459175400186208261?s=20](https://twitter.com/bertcmiller/status/1459175400186208261?s=20)
# **Token Integration Issues in UniswapV3**

In the world of decentralized exchanges like UniswapV3, keeping things secure is super important. Right now, there are some problems with certain kinds of tokens, specifically Fee-on-transfer and rebasing tokens, when it comes to using them on UniswapV3.

### **Fee-on-transfer Tokens**

So, Fee-on-transfer tokens, which add a little fee with each transaction, aren’t working smoothly with UniswapV3’s router contracts. It’s like they’re not talking to each other properly. To fix this, the smart folks creating these tokens might have to find a different way, like making a special wrapper or a customized router. But here’s the thing — UniswapV3 won’t be creating a router that supports these tokens in the future. So, the creators need to figure it out on their own.

### **Rebasing Tokens**

Now, let’s talk about rebasing tokens. These are a bit different because they change the amount of tokens in circulation based on some rules. They’re good for creating and swapping in pools, but there’s a catch. If you’re putting your money into a pool and there’s a negative rebase (meaning you lose some tokens), there’s no way to get back what you lost. Liquidity providers, the people putting their money in, need to be careful because they might face losses with no chance of recovering them.

# **Recommendations when auditing**

1. Use a general [SCSVS](https://securing.github.io/SCSVS/) checklist.
    
2. Validate for correct implementation of [CEI pattern](https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html) in all contracts: check conditions, then make changes, and then interact with other contracts.
    
3. Review all cases related to reentrancy vulnerabilities. Even if all contracts are under nonReentrant modifier — check for read only reentrancy.
    
4. Check the Solidity version for common compiler bugs, check the coverage of unit testing and if there are fuzzing tests.
    
5. Validate that project uses well-tested and secure mathematical libraries.
    
6. Validate that math in contracts works exactly the same as it was declared in the documentation.
    
7. Check how math works with small numbers of decimals as well as with big numbers of decimals.
    
8. Compare the protocol with its fork, or the protocol from which it was inspired, check for common integration issues, and for new features, missing validations.
    
9. Check all mathematical calculations for rounding issues, overflows and underflows.
    
10. Validate if all contracts have proper validation of input params, be careful with slippage, deadline, etc. The protocol should use SafeErc20 for token interactions.
    
11. Check access control of all contracts.
    
12. Validate cases with non-standard tokens.
    
13. Check the possibilities of price manipulations in the pool.
    
14. Validate that roles are not overpowered, protocol uses multisig for managing the contracts.
    
15. Check that fee is always calculated correctly and can never be 100%.
    
16. Check that in case the protocol has an oracle — it’s built on the architecture of TWAP.
    
17. Validate all the calls to other contracts, pending allowance.
# **Summary**

In this article, we have classified the main security vectors encountered by automated trading protocols in recent years. Paying special attention to these vectors is a crucial aspect during the auditing process.
