### [H-1] Looping with for loop can leads to gas limit and cause DOS attack

**Description:** DOS attack is a case of servive denial which could break the protocol functionality. There are varying ways this can occure and this right here is one of them. The `PuppyRaffle::enterRaffle` function loops through the players array to check for duplicates. The longer the array of players, the more checks to make.

**Impact:** If a user sybil with many accounts, this causes increase in gas and discourages others to participate and give this user higher chance of winning.

**Proof of Concept:** 
Run this test and observe the gasLimit;
- With looping: gas: 339880
- With Enum+Mapping: 321867
- 18,013 gas difference
  
forge test-
```javascript
function testCanEnterRaffle() public {
        address[] memory players = new address[](10);
        players[0] = playerOne;
        players[1] = playerTwo;
        players[2] = playerThree;
        players[3] = playerFour;
        players[4] = address(5);
        players[5] = address(6);
        players[6] = address(7);
        players[7] = address(8);
        players[8] = address(9);
        players[9] = address(10);
        puppyRaffle.enterRaffle{value: entranceFee * 10}(players);
        assertEq(puppyRaffle.players(0), playerOne);
        assertEq(puppyRaffle.players(1), playerTwo);
        assertEq(puppyRaffle.players(2), playerThree);
        assertEq(puppyRaffle.players(3), playerFour);
        assertEq(puppyRaffle.players(4), address(5));
        assertEq(puppyRaffle.players(5), address(6));
        assertEq(puppyRaffle.players(6), address(7));
        assertEq(puppyRaffle.players(7), address(8));
        assertEq(puppyRaffle.players(8), address(9));
        assertEq(puppyRaffle.players(9), address(10));
    }
```
Subtitute for loop for status check to spot the difference gas used.

**Mitigation:** Use enum status like ENTERED, NON-PARTICIPANTS or boolean expression to mitigate the need to loop before confirming if user has already entered the raffle or not.
- Create ENUM;
  ```diff
  + enum EnterRaffleStatus { NOT_ENTERED, ENTERED }
    ```
- Create a mapping with the enum;
  ```diff
  + mapping(address => EnterRaffleStatus) public playerEnteredStatus;
  ```

- Check the user status and Update after raffle is entered;
  ```javascript
  if (playerEnteredStatus[msg.sender] == EnterRaffleStatus.ENTERED) {
            revert("PuppyRaffle: Duplicate player");
        }
  ```



## Likelihood and Impact:
- Impact: HIGH
- Likelihood: MEDIUM
- Severity: HIGH


### [M-1] For gas efficiency, newPlayers.length should be assigned to a variable. may cause DOS attack

**Description:** The `PuppyRaffle::enterRaffle` has length of array to loop through and this looping increases gas potentially.

**Impact:** This may lead to gas increae and discourages others from participating. This alsoincreases the costof deploying the protocol.

**Proof of Concept:** 
This is a very much known concept.

**Mitigation:**
- Assign the length to a uint256 variable and subtitute it
  ```diff
  + uint256 newPlayersLength = newPlayers.length;
  ```


## Likelihood and Impact:
- Impact: HIGH
- Likelihood: HIGH
- Severity: MEDIUM

### [] Integer overflow

**Description:** There is overflow vulnerability here since totalFees is a uint64, and fee could be a large number if there are many players.

**Impact:** This could cause totalFees to overflow and wrap around to zero, which would allow anyone to withdraw all the funds from the contract.

**Proof of Concept:**
```javascript
function testTotalFeesOverflow() public {
        address[] memory players = new address[](100);
        for (uint256 i = 0; i < 100; i++) {
            players[i] = address(i + 1);
        }
        puppyRaffle.enterRaffle{value: entranceFee * 100}(players);

        vm.warp(block.timestamp + duration + 1);
        vm.roll(block.number + 1);

        puppyRaffle.selectWinner();

        uint256 expectedTotalFees = ((entranceFee * 100) * 20) / 100;
        assertEq(puppyRaffle.totalFees(), expectedTotalFees);
    }
```

**Mitigation:** We could change totalFees to a uint256, which would allow for a much larger number of players without risking overflow.
```diff
- totalFees = totalFees + uint64(fee);
```
```diff
+ totalFees = totalFees + uint256(fee);
```
### [] Unsafe Casting

**Description:** `puppyRaffle::selectWinner` function calculates the fee and adds it to totalFees, but it casts the fee to uint64 before adding it to totalFees, which is a uint256. If the fee is larger than the maximum value of uint64, it will overflow and wrap around to zero.

**Impact:** This could cause totalFees to overflow and wrap around to zero, which would allow anyone to withdraw all the funds from the contract.

**Proof of Concept:**
```javascript
contract Unsafe{
    uint64 private _unSafe1 = type(uint64).max;

    function check_value() public view{
        _unSafe1;
    }
    function change_value() public {
        _unSafe1 = uint256(20);
    }
}
```
Here, an error is thrown when we try to change the value of _unSafe1 to 20, because it is being cast to uint64, which cannot hold the value of 20.
*TypeError: Type uint256 is not implicitly convertible to expected type uint64.
  --> unsafe.sol:11:20:
   |
11 |         _unSafe1 = uint256(20); //Unsafe casting
   |                    ^^^^^^^^^^^
*

**Mitigation:** Uncast the fee and let it be added to totalFees as a uint256, which would allow for a much larger number of players without risking overflow.
```diff
- totalFees = totalFees + uint64(fee);
```
```diff
+ totalFees = totalFees + fee;
```

### [H-1] Mishandling of eth
**Description:** The `PuppyRaffle::withdrawFees` function checks if the balance of the contract is equal to totalFees before allowing the owner to withdraw the fees. However, this check can be bypassed if an attacker sends a large amount of ether to the contract, which would increase the balance and allow them to withdraw the fees even if there are still players in the raffle. This could lead to a situation where the owner is unable to withdraw the fees, which could cause financial loss for the owner and damage the reputation of the contract.
```diff
            address(this).balance == uint256(totalFees),
            "PuppyRaffle: There are currently players active!"
```

**Impact:** This condition is meant to prevent the owner from withdrawing fees while there are still players in the raffle, but it can be bypassed by anyone who can manipulate the balance of the contract. For example, an attacker could send a large amount of ether to the contract, which would increase the balance and allow them to withdraw the fees even if there are still players in the raffle. This could lead to a situation where the owner is unable to withdraw the fees, which could cause financial loss for the owner and damage the reputation of the contract.

**Proof of Concept:**
1. An attacker could deploy a malicious contract that sends a large amount of ether to the PuppyRaffle contract.
```javascript
contract Malicious {
    function attack(address payable _puppyRaffle) public payable {
        require(msg.value > 0, "Must send some ether");
        (bool success, ) = _puppyRaffle.call{value: msg.value}("");
        require(success, "Attack failed");
    }
}
```
**Mitigation:** To mitigate this issue, the `withdrawFees` function should be restricted to only the owner of the contract, and the check for the balance should be removed. Instead, the function should simply allow the owner to withdraw the fees without any conditions. This way, even if an attacker sends a large amount of ether to the contract, it will not affect the owner's ability to withdraw the fees.
