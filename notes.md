Q
1. /// q Why do players have to pay entrance fee * no of players? Line 82
2. /// s For gas efficiency, newPlayers.length should be assigned to a variable. Line 83
3. /// s Use enum status like ENTERED, NON-PARTICIPANTS. Line 89
4. /// q What does this do exactly?
        require(playerAddress != address(0), "PuppyRaffle: Player already refunded, or is not active");
5. /// e Function should be payable since money is being retrieved. Line 106
6. ///q This will eventually return zero NO? should be Else {return 0}. Line 122
7. /// q Anyone can withdraw fees and also function should be payable. line 164