# Blockchain & Solidity Vulnerabilities

Adding a version pragma is a best practice, as it avoids problems with mismatched compiler and language versions. 

#### Avoid Dynamically Sized Arrays

Any loop through a dynamically sized array where a function performs operations on each element or searches for a particular element introduces the risk of using too much gas. Indeed, the contract may run out of gas before finding the desired result, or before acting on every element, thus wasting time and ether without giving any result at all.

#### Avoid Calls to Other Contracts

Calling other contracts, especially when the gas cost of their functions is not known, introduces the risk of running out of gas. Avoid using libraries that are not well tested and broadly used. The less scrutiny a library has received from other programmers, the greater the risk of using it.

### Protecting Against Overflow Errors at the Compiler Level

Overflow errors in software can be catastrophic when dealing with real value. For example, one [transaction from mid-April 2018](http://bit.ly/2yHfvoF) shows the malicious transfer of over 57,896,044,618,658,100,000,000,000,000,000,000,000,000, 000,000,000,000,000,000 BEC tokens. This transaction was the result of an integer overflow issue in BeautyChain’s ERC20 token contract (*BecToken.sol*). Solidity developers do have access to libraries like [SafeMath](http://bit.ly/2ABhb4l) as well as Ethereum smart contract security analysis tools like [Mythril OSS](http://bit.ly/2CQRoGU). However, developers are not forced to use the safety tools. Put simply, if safety is not enforced by the language, developers can write unsafe code that will successfully compile and later on "successfully" execute.

Vyper has built-in overflow protection, implemented in a two-pronged approach. Firstly, Vyper provides [a SafeMath equivalent](http://bit.ly/2PuDfpB) that includes the necessary exception cases for integer arithmetic. Secondly, Vyper uses clamps whenever a literal constant is loaded, a value is passed to a function, or a variable is assigned. Clamps are implemented via custom functions in the Low-level Lisp-like Language (LLL) compiler, and cannot be disabled. (The Vyper compiler outputs LLL rather than EVM bytecode; this simplifies the development of Vyper itself.)

### Transaction Gas

Gas is the fuel of Ethereum. Gas is not ether—it’s a separate virtual currency with its own exchange rate against ether. Ethereum uses gas to control the amount of resources that a transaction can use, since it will be processed on thousands of computers around the world. The open-ended (Turing-complete) computation model requires some form of metering in order to avoid denial-of-service attacks or inadvertently resource-devouring transactions.

#### Estimating Gas Cost

If you need to estimate the gas necessary to execute a certain method of a contract considering its arguments, you could use the following procedure:

```c++
var contract = web3.eth.contract(abi).at(address);
var gasEstimate = contract.myAweSomeMethod.estimateGas(arg1, arg2,
    {from: account});
```

gasEstimate will tell you the number of gas units needed for its execution. It is an estimate because of the Turing completeness of the EVM—it is relatively trivial to create a function that will take vastly different amounts of gas to execute different calls. Even production code can change execution paths in subtle ways, resulting in hugely different gas costs from one call to the next. However, most functions are sensible and estimateGas will give a good estimate most of the time.

To obtain the gas price from the network you can use:

```c++
var gasPrice = web3.eth.getGasPrice();
```

And from there you can estimate the gas cost:

```
var gasCostInEther = web3.utils.fromWei((gasEstimate * gasPrice), 'ether');
```

