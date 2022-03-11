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


