# Staker Contract to stake ETH for group funding effort
I developed the contract for a decentralized staker contract that can be used for group funding efforts. This is the solution for the challenge 1 of Scaffold Eth - creating a token vendor. This particular code passes all the given 6 tests and has been accepted by speedrunethereum.

> _Please do not copy this code as is, I suggest you work on your own solution and if necessary, take inspiration from the code that I've written._

The following links will help you find my contracts and transactions:
- Staker Contract Code - [here](https://goerli.etherscan.io/address/0xf6BCDaFf77A1829de4eE60fb813F379ea7b8ae6B#code)
- Deployed at this link - http://sordid-card.surge.sh/


> ```solidity 
> mapping (address => uint256) public balances;
> ```
Let's make a mapping of all the sender addresses to numbers uint and name it balances. This mapping allows us to track user's balances.

> ```solidity 
> uint256 public constant threshold = 1 ether;
> ```
Additionally let's maintain a threshold of 1 ether so that stake can execute when the threshold or higher amount of eth balance in the contract is present.

## stake() function
```solidity

  contract Staker {

  ExampleExternalContract public exampleExternalContract;

  constructor(address exampleExternalContractAddress) {
      exampleExternalContract = ExampleExternalContract(exampleExternalContractAddress);
  }

  mapping (address => uint256) public balances;

  // Declaring a constant threshold of 1 ether
  uint256 public constant threshold = 1 ether;

  event Stake(address indexed sender, uint256 amount);

  // Collect funds in a payable `stake()` function and track individual `balances` with a mapping:
  // ( Make sure to add a `Stake(address,uint256)` event and emit it for the frontend <List/> display )
  function stake() public payable{
    balances[msg.sender] += msg.value;
    emit Stake(msg.sender, msg.value);
  }

```
The stake function adds the balance of the user and the amount of value they pay into our mapping of ```balances```. This function then emits the event. The ```event Stake(address indexed sender, uint256 amount);``` declared before this function allows us to emit the changes to the frontend.

> ```solidity 
> uint256 public deadline = block.timestamp + 72 hours;
> ```
Now let's declare a deadline like this so that users can execute and withdraw from the stake when the deadline is passed.

## execute() function
```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.4;  //Do not change the solidity version as it negativly impacts submission grading

import "hardhat/console.sol";
import "./ExampleExternalContract.sol";

contract Staker {

  ExampleExternalContract public exampleExternalContract;

  constructor(address exampleExternalContractAddress) {
      exampleExternalContract = ExampleExternalContract(exampleExternalContractAddress);
  }

  mapping (address => uint256) public balances;

  // Declaring a constant threshold of 1 ether
  uint256 public constant threshold = 1 ether;

  event Stake(address indexed sender, uint256 amount);

  // Collect funds in a payable `stake()` function and track individual `balances` with a mapping:
  // ( Make sure to add a `Stake(address,uint256)` event and emit it for the frontend <List/> display )
  function stake() public payable{
    balances[msg.sender] += msg.value;
    emit Stake(msg.sender, msg.value);
  }

  // After some `deadline` allow anyone to call an `execute()` function
  // If the deadline has passed and the threshold is met, it should call `exampleExternalContract.complete{value: address(this).balance}()`
  bool public openForWithdraw = true;

  uint256 public deadline = block.timestamp + 72 hours;

  modifier deadlineExpired() {
    require(block.timestamp >= deadline, "Deadline not reached, please wait for timer to hit 0");
    _;
  }

  modifier notCompleted() {
    bool completed = exampleExternalContract.completed();
    require(!completed, "staking process already completed");
    _;
  }

  function execute() public deadlineExpired notCompleted{
    // deadline should be lesser than the threshold   
    //exampleExternalContract.complete{value: address(this).balance}();\\

    if(address(this).balance >= threshold){
      (bool sent, ) = address(exampleExternalContract).call{value: address(this).balance}(abi.encodeWithSignature("complete()"));
      require(sent, "Stake incomplete");
    }
  }
}
```

In this snippet, you can see that we have 2 modifers declared - notCompleted() and deadlineExpired(). Function modifiers are used to modify the behaviour of a function. Best part is it can be reusable multiple times throughout your code.

The first modifier ```deadlineExpired()``` can check if the deadline has been expired or not.
The second modifier ```notCompleted()``` will check if the staking process is completed or not.

The execute() function first requires the staking to not be completed and the deadline to not have passed. Once that is done, we check if the balance in the contract is greater than the 1 ether threshold or not. If the balance of the contract is greater than threshold, the staking completion value can be marked as completed.


## withdraw()

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.4;  //Do not change the solidity version as it negativly impacts submission grading

import "hardhat/console.sol";
import "./ExampleExternalContract.sol";

contract Staker {

  ExampleExternalContract public exampleExternalContract;

  constructor(address exampleExternalContractAddress) {
      exampleExternalContract = ExampleExternalContract(exampleExternalContractAddress);
  }

  mapping (address => uint256) public balances;

  // Declaring a constant threshold of 1 ether
  uint256 public constant threshold = 1 ether;

  event Stake(address indexed sender, uint256 amount);

  // Collect funds in a payable `stake()` function and track individual `balances` with a mapping:
  // ( Make sure to add a `Stake(address,uint256)` event and emit it for the frontend <List/> display )
  function stake() public payable{
    balances[msg.sender] += msg.value;
    emit Stake(msg.sender, msg.value);
  }

  // After some `deadline` allow anyone to call an `execute()` function
  // If the deadline has passed and the threshold is met, it should call `exampleExternalContract.complete{value: address(this).balance}()`
  bool public openForWithdraw = true;

  uint256 public deadline = block.timestamp + 72 hours;

  modifier deadlineExpired() {
    require(block.timestamp >= deadline, "Deadline not reached, please wait for timer to hit 0");
    _;
  }

  modifier notCompleted() {
    bool completed = exampleExternalContract.completed();
    require(!completed, "staking process already completed");
    _;
  }

  function execute() public deadlineExpired notCompleted{
    // deadline should be lesser than the threshold
    // If the address(this).balance of the contract is over the threshold by the deadline, 
    // you will want to call: exampleExternalContract.complete{value: address(this).balance}()
    //require(address(this).balance <= threshold, "Lesser money than threshold");
   
    //exampleExternalContract.complete{value: address(this).balance}();\\

    if(address(this).balance > threshold){
      (bool sent, ) = address(exampleExternalContract).call{value: address(this).balance}(abi.encodeWithSignature("complete()"));
      require(sent, "Stake incomplete");
    }
  }

  // If the `threshold` was not met, allow everyone to call a `withdraw()` function to withdraw their balance
  function withdraw() public notCompleted{
    //require(openForWithdraw == true, "Not open for withdraw");
    require(balances[msg.sender] > 0, "No balance in sender's account");
    balances[msg.sender] = 0;

    (bool sent, ) = msg.sender.call{value: address(this).balance}("");
    require(sent, "Failed to send user balance back to the user");
  }
}
```

In order for withdraw to work, the stake should not be completed and we need to ensure that the user has balance in his account. Else, everything will be sent back to the user. Again, if you remember all these balances are stored in the mapping that we declared early on.


## timeLeft() function

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.4;  //Do not change the solidity version as it negativly impacts submission grading

import "hardhat/console.sol";
import "./ExampleExternalContract.sol";

contract Staker {

  ExampleExternalContract public exampleExternalContract;

  constructor(address exampleExternalContractAddress) {
      exampleExternalContract = ExampleExternalContract(exampleExternalContractAddress);
  }

  mapping (address => uint256) public balances;

  // Declaring a constant threshold of 1 ether
  uint256 public constant threshold = 1 ether;

  event Stake(address indexed sender, uint256 amount);

  // Collect funds in a payable `stake()` function and track individual `balances` with a mapping:
  // ( Make sure to add a `Stake(address,uint256)` event and emit it for the frontend <List/> display )
  function stake() public payable{
    balances[msg.sender] += msg.value;
    emit Stake(msg.sender, msg.value);
  }

  // After some `deadline` allow anyone to call an `execute()` function
  // If the deadline has passed and the threshold is met, it should call `exampleExternalContract.complete{value: address(this).balance}()`
  bool public openForWithdraw = true;

  uint256 public deadline = block.timestamp + 72 hours;

  modifier deadlineExpired() {
    require(block.timestamp >= deadline, "Deadline not reached, please wait for timer to hit 0");
    _;
  }

  modifier notCompleted() {
    bool completed = exampleExternalContract.completed();
    require(!completed, "staking process already completed");
    _;
  }

  function execute() public deadlineExpired notCompleted{
    // deadline should be lesser than the threshold
    // If the address(this).balance of the contract is over the threshold by the deadline, 
    // you will want to call: exampleExternalContract.complete{value: address(this).balance}()
    //require(address(this).balance <= threshold, "Lesser money than threshold");
   
    //exampleExternalContract.complete{value: address(this).balance}();\\

    if(address(this).balance > threshold){
      (bool sent, ) = address(exampleExternalContract).call{value: address(this).balance}(abi.encodeWithSignature("complete()"));
      require(sent, "Stake incomplete");
    }
  }

  // If the `threshold` was not met, allow everyone to call a `withdraw()` function to withdraw their balance
  function withdraw() public notCompleted{
    //require(openForWithdraw == true, "Not open for withdraw");
    require(balances[msg.sender] > 0, "No balance in sender's account");
    balances[msg.sender] = 0;

    (bool sent, ) = msg.sender.call{value: address(this).balance}("");
    require(sent, "Failed to send user balance back to the user");
  }


  // Add a `timeLeft()` view function that returns the time left before the deadline for the frontend
  function timeLeft() public view returns (uint256){
    if(block.timestamp >= deadline){
      return 0;
    }else{
      return deadline - block.timestamp;
    }
  }

}
```
The timeLeft() function is supposed to display the time left for the deadline to finish. 

That's it! That's all the code that passes all tests. If you enjoyed that, and want to show me some support, you can send me ETH at 0x04b24656E4B114e4eF83f40a1161d1804e684D89.
