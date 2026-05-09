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