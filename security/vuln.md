# Blockchain & Solidity Vulnerabilities

**<u>Table of Contents:</u>**

[Security Best Practices](#Security-Best-Practices)



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

### Security Best Practices

*Defensive programming* is a style of programming that is particularly well suited to smart contracts. It emphasizes the following, all of which are best practices:

- Minimalism/simplicity

  Complexity is the enemy of security. The simpler the code, and the less it does, the lower the chances are of a bug or unforeseen effect occurring. When first engaging in smart contract programming, developers are often tempted to try to write a lot of code. Instead, you should look through your smart contract code and try to find ways to do less, with fewer lines of code, less complexity, and fewer "features." If someone tells you that their project has produced "thousands of lines of code" for their smart contracts, you should question the security of that project. Simpler is more secure.

- Code reuse

  Try not to reinvent the wheel. If a library or contract already exists that does most of what you need, reuse it. Within your own code, follow the DRY principle: Don’t Repeat Yourself. If you see any snippet of code repeated more than once, ask yourself whether it could be written as a function or library and reused. Code that has been extensively used and tested is likely more secure than any new code you write. Beware of “Not Invented Here” syndrome, where you are tempted to "improve" a feature or component by building it from scratch. The security risk is often greater than the improvement value.

- Code quality

  Smart contract code is unforgiving. Every bug can lead to monetary loss. You should not treat smart contract programming the same way as general-purpose programming. Writing DApps in Solidity is not like creating a web widget in JavaScript. Rather, you should apply rigorous engineering and software development methodologies, as you would in aerospace engineering or any similarly unforgiving discipline. Once you "launch" your code, there’s little you can do to fix any problems.

- Readability/auditability

  Your code should be clear and easy to comprehend. The easier it is to read, the easier it is to audit. Smart contracts are public, as everyone can read the bytecode and anyone can reverse-engineer it. Therefore, it is beneficial to develop your work in public, using collaborative and open source methodologies, to draw upon the collective wisdom of the developer community and benefit from the highest common denominator of open source development. You should write code that is well documented and easy to read, following the style and naming conventions that are part of the Ethereum community.

- Test coverage

  Test everything that you can. Smart contracts run in a public execution environment, where anyone can execute them with whatever input they want. You should never assume that input, such as function arguments, is well formed, properly bounded, or has a benign purpose. Test all arguments to make sure they are within expected ranges and properly formatted before allowing execution of your code to continue.