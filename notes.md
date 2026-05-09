Q

3. /// s Use enum status like ENTERED, NON-PARTICIPANTS. Line 89
   
4. /// @audit This function is vulnerable to reentrancy attacks, since the state is updated after the call to the player. To fix this, we should update the state before sending the funds.
    /// q Why do we need playerIndex? Why not just msg.sender?
    
5. /// q What does this do exactly?
        require(playerAddress != address(0), "PuppyRaffle: Player already refunded, or is not active");

6. /// e Function should be payable since money is being retrieved. Line 106


7. ///q This will eventually return zero NO? should be Else {return 0}. Line 122
8. /// q Anyone can withdraw fees and also function should be payable. line 164
9.  /// q Why do players have to pay entrance fee * no of players? Line 82
    
10. /// s For gas efficiency, newPlayers.length should be assigned to a variable. Line 83