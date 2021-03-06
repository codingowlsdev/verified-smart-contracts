*2018-10-12*

# Lightweight Formal Verification of Uniswap Smart Contract

￼We present a *lightweight* formal verification of the [Uniswap] smart contract, a decentralized token exchange contract, specifically [uniswap_exchange.vy] (of git-commit-id [cb43082]).

Our lightweight verification consists of two parts:
 1. Formalization of the "[`x*y=k`]" market maker model and its smart contract implementation.
 1. Symbolic execution of the compiled [bytecode] of four important functions.

## Part I: Formalization of `x*y=k` model and its implementation

We formalize the `x*y=k` market maker model and its smart contract implementation.

Note that there exists a gap between the mathematical model and its implementation, since the implementation does not use real arithmetic but integer arithmetic approximating real numbers by rounding.  Thus, we also formally analyze the approximation error caused by the integer rounding, showing that the error is bounded and does not lead to violation of the critical properties denoted by the mathematical model.  (*Indeed, we found that the previous implementation did not satisfy the desired properties due to incorrect handling of rounding errors, which is fixed in the current version [cb43082] as [we suggested][issues.md].*)

The formalization is presented in the companion document [x-y-k.pdf].

## Part II: Symbolic execution of bytecode

We mechanically explored, by running the [KEVM] symbolic execution engine, all possible behaviors of the compiled [bytecode] of the four important functions, [`addLiquidity`], [`removeLiquidity`], [`ethToTokenSwapInput`], and [`ethToTokenSwapOutput`], to ensure that the bytecode does not introduce any unexpected behavior other than what is specified in the source code.  Essentially, this ensures that no compiler bug is involved in generating the bytecode from the current source code.

Here we abstract the external token contract calls (i.e., `self.token.balanceOf`, `self.token.transfer`, and `self.token.transferFrom`) that are unknown, by assuming that the external token contract is legitimate, that is, that it conforms to the ERC20 standard (e.g., it MUST return true when it succeeds), and it does not alter the exchange contract state (i.e., the storage).  This assumption is considered reasonable in the sense that it is the user's responsibility to create an exchange on a legitimate token contract.  *We strongly recommend a client to formally verify the token contract before creating an exchange.  Refer to [our formal verification of ERC20 token contracts][erc20] for more details.*

Since this is a *lightweight* formal analysis engagement, we omit the gas cost analysis in our symbolic analysis.  (Note that, however, full formal verification engagements will formally specify and verify the exact gas consumption of each function, as a symbolic expression of its input.)

The [bytecode] is obtained by using the Vyper compiler of git-commit-id [35038d2].  Note that the target [bytecode] is the *runtime bytecode* (generated by `vyper -f bytecode_runtime`) that is actually executed when running an exchange contract, which is *different* from the *[contract creation bytecode]* (generated by `vyper -f bytecode`) that is used for creating an exchange contract.

The symbolic execution results are provided in the [results] directory.  For each function, the symbolic execution result presents a set of symbolic output states along with a corresponding path-condition that captures all possible execution paths of the bytecode.

#### Reproducing results

* Install [K] prerequisites. No need to install K itself as scripts below will do it.
* Install dependencies
```sh
$ make -C uniswap deps
```

* Generate specifications
```sh
$ make -C uniswap split-proof-tests
```

* Produce symbolic execution results
```sh
$ make -C uniswap test
```

See these [instructions] for more details of running the verifier.

## Issues found

During the verification process, we found several issues, and suggested fixes.  All issues we found are fixed in the current version [cb43082].  See [issues.md] for details.

## [Resources](../README.md#resources)

## [Disclaimer](../README.md#disclaimer)

*Note that, compared to full verification, the lightweight verification would not provide the full end-to-end guarantee for the security properties. The provided proof is not fully mechanized, but part of the proof is provided in a pencil-and-paper form. Also, it would not provide the full correctness of the entire compilation but prove the correctness of compilation for only certain critical features. This incompleteness of the lightweight verification could result in missing some potential security vulnerabilities.*




[Uniswap]: <https://github.com/Uniswap/contracts-vyper/tree/cb4308226f07cafa445b2255b01d148e7ab6af9f>
[cb43082]: <https://github.com/Uniswap/contracts-vyper/commits/cb4308226f07cafa445b2255b01d148e7ab6af9f>
[uniswap_exchange.vy]: <https://github.com/Uniswap/contracts-vyper/blob/cb4308226f07cafa445b2255b01d148e7ab6af9f/contracts/uniswap_exchange.vy>
[x-y-k.pdf]: </uniswap/x-y-k.pdf>
[issues.md]: </uniswap/issues.md>
[results]: </uniswap/results>

[bytecode]: <https://github.com/runtimeverification/verified-smart-contracts/blob/uniswap/uniswap/code/bytes.txt>
[contract creation bytecode]: <https://github.com/Uniswap/contracts-vyper/blob/cb4308226f07cafa445b2255b01d148e7ab6af9f/bytecode/exchange.txt>

[`x*y=k`]: <https://ethresear.ch/t/improving-front-running-resistance-of-x-y-k-market-makers>

[`addLiquidity`]: <https://github.com/Uniswap/contracts-vyper/blob/cb4308226f07cafa445b2255b01d148e7ab6af9f/contracts/uniswap_exchange.vy#L41-L75>
[`removeLiquidity`]: <https://github.com/Uniswap/contracts-vyper/blob/cb4308226f07cafa445b2255b01d148e7ab6af9f/contracts/uniswap_exchange.vy#L77-L98>
[`ethToTokenSwapInput`]: <https://github.com/Uniswap/contracts-vyper/blob/cb4308226f07cafa445b2255b01d148e7ab6af9f/contracts/uniswap_exchange.vy#L145-L153>
[`ethToTokenSwapOutput`]: <https://github.com/Uniswap/contracts-vyper/blob/cb4308226f07cafa445b2255b01d148e7ab6af9f/contracts/uniswap_exchange.vy#L180-L188>

[K]: <https://github.com/kframework/k>
[KEVM]: <https://github.com/kframework/evm-semantics/tree/81bcba1a73188942fdc34979c59ee7be7253ece9>
[instructions]: </resources/instructions.md>
[erc20]: </erc20/README.md>

[35038d2]: <https://github.com/ethereum/vyper/commits/35038d20bd9946a35261c4c4fbcb27fe61e65f78>
