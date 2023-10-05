Oblong Clay Kangaroo

high

# CREATE3 is not available in the zkSync Era.
In the current code using CREATE3 with CREATE2, but in zkSync Era, CREATE2 for arbitrary bytecode is not available, so a revert occurs in the `CREATE3.deploy` process.

## Vulnerability Detail
According to the contest README, the project can be deployed in zkSync Era. (https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/README.md?plain=1#L11)

The zkSync Era docs explain how it differs from Ethereum.

The description of CREATE and CREATE2 (https://era.zksync.io/docs/reference/architecture/differences-with-ethereum.html#create-create2) states that Create cannot be used for arbitrary code unknown to the compiler.

POC: 
```solidity
// SPDX-License-Identifier: Unlicensed
pragma solidity ^0.8.0;

import "./MiniContract.sol";
import "./CREATE3.sol";

contract DeployTest {
    address public deployedAddress;
    event Deployed(address);
    
    function generateContract() public returns(address, address) {
        bytes32 salt = keccak256("SALT");

        address preCalculatedAddress = CREATE3.getDeployed(salt);

        // check if the contract has already been deployed by checking code size of address
        bytes memory creationCode = abi.encodePacked(type(MiniContract).creationCode, abi.encode(777));

        // Use CREATE3 to deploy the anchor contract
        address deployed = CREATE3.deploy(salt, creationCode, 0);
        return (preCalculatedAddress, deployed);
    }
}
```
You can check sample POC code at zkSync Era Testnet(https://goerli.explorer.zksync.io/address/0x0f670f8AfcB09f4BC509Cb59D6e7CEC1A52BFA51#contract)

Also, the logic to compute the address of Create2 is different from Ethereum, as shown below, so the CREATE3 library cannot be used as it is.

This cause registry returns an incorrect `preCalculatedAddress`, causing the anchor to be registered to an address that is not the actual deployed address.

```solidity 
address ⇒ keccak256( 
    keccak256("zksyncCreate2") ⇒ 0x2020dba91b30cc0006188af794c2fb30dd8520db7e2c088b7fc7c103c00ca494, 
    sender, 
    salt, 
    keccak256(bytecode), 
    keccak256(constructorInput)
 ) 
```



## Impact
`generateAnchor` doesn't work, so user can't do anything related to anchor.

## Code Snippet
https://github.com/allo-protocol/allo-v2/blob/851571c27df5c16f6586ece2a1cb6fd0acf04ec9/contracts/core/Registry.sol#L350
https://github.com/allo-protocol/allo-v2/blob/851571c27df5c16f6586ece2a1cb6fd0acf04ec9/contracts/core/Registry.sol#L338
## Tool used

Manual Review

## Recommendation
This can be solved by implementing CREATE2 directly instead of CREATE3 and using `type(Anchor).creationCode`.
Also, the compute address logic needs to be modified for zkSync.