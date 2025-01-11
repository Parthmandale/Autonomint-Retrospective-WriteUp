# Autonomint-Retrospective-WriteUp

## Approach 
- Core problem:
- What steps lead the auditor to find this bug?
- What question he posed that lead him to this bug?
- How to find it next time:

## [H-1-696] - Users can withdraw liquidated collateral (through liq type 2) 

#### Core problem:
- This `depositDetail.liquidated` gets updated in liq type 1 but not in type 2, due to this in withdraw function this line `if (depositDetail.liquidated) revert IBorrowing.Borrow_AlreadyLiquidated();` will not revert for for liq type 2, and due to this user will be able to repay his debt and take out his collateral even after
he hot liquidated. Now when liq type2 happened at that time collateral of borrower was already sent to synthetics, but now when he is withdrawing the collateral,
here it means he is withdrawing someone elses collateral, its like stealing the funds of somone else, making loss of funds to other borrower, and even protocol.

- Also another problem is that a big amount of state changes did not occured in liq type2, that were occured in typ1 [here](https://github.com/sherlock-audit/2024-11-autonomint/blob/0d324e04d4c0ca306e1ae4d4c65f0cb9d681751b/Blockchain/Blockchian/contracts/Core_logic/borrowLiquidation.sol#L241-L277). This inconsistency will create a bad state for protocol. 

#### What steps lead the auditor to find this bug?
- He simply compared the difference between liq type 1 and type 2, that's it.
 
#### What question he posed that lead him to this bug?
- When he saw that in liq type 2 `depositDetail.liquidated` state of borrower is not updated, he searched the exact state changes state in all contracts, and found a bad state impact in function withdraw 

#### How to find it next time:
- Simply compare the two likely/opposite functions, and compare there state changes and if you found some inconsistency then try to see the bad state due to that inconsistency.


## [H-2-272] - Borrowing::redeemYields debits ABOND from msg.sender but redeems to user using ABOND.State data from user

#### Core problem:
- `msg.sender` and `user` for which the redeem is supposed to be done, should have been been same, with a check, but its missing. And due to this, many inconsistency is occuring and therefore this bug is giving path to attack vector.
- user state here `State memory userState = abond.userStates(user);` is not updated at all, dev assumed that state will be updated in here - `bool success = abond.burnFromUser(msg.sender, aBondAmount);` but here state of `msg.sender` is getting updated, and here msg.sender and user address can be different.
- So attacker use different address where he bought a lot of ABOND token from market, and then execute this attack, as real user's state is not updated at all, msg.sender's ABOND token will be burned, but still loss of ETH is happening to protocol, as all eth that were deposited in external protocol are no removed from there, and is gone to `user` account (which is also of an attacker).
- another Impact will be that price of ABOND token will fall significantly, due this attack.

#### What steps lead the auditor to find this bug?
- he noticed that no check for msg.sender and user to be same is implemented and on top of that while reedeming user's address is been used and while burning msg.sender's address is been used. After this he tried to think about attack vector to exploit this inconsistency.

#### What question he posed that lead him to this bug?
- what if msg.sender and user address is different, then will this function be still executable, without any revert or not, and if not then he tried to bring bad state for it.

#### How to find it next time:
- next time you see this kind of inconsistency, take an example where msg.sender and user are different with in reality both address holding by an attacker and try to exploit by taking different kind of scenarios, like in this one msg.sender address of attacker didn't had any ABOND token, but still just to apply this attack he bought it from external market. So apply all this kind of logic in order to bring the bad state of protocol and other users.

## [H-3-746] - Type 1 borrower liquidation will incorrectly add cds profit directly to totalCdsDepositedAmount

#### Core problem:
-  incorrect reduction by `omniChainData.totalCdsDepositedAmount -= liquidationAmountNeeded - cdsProfits;` The problem with this approach is that it will always calculate cumulative values and option fees based on this value,  which now differs from the individual sum of cds depositors, leading to untracked values, such as the cumulative value from the vault long position or option fees.
-  impact - Cumulative value calculations and option fees will be incorrect, as they are divided by a bigger number of cds deposited (which was added the profit), but each cds depositor only has the same deposited amount to multiply by these rates.

#### What steps lead the auditor to find this bug?
- he mainly found this because he knew exactly what is `omniChainData.totalCdsDepositedAmount` and where and how it was supposed to be updated and by how much, he knew if this got calculated then other things that are calculated on the basis of it, will also be incorrectly calculated and will create problem. So he knew this was incorrrectly calculated then he found the impact by navigating calculations where `omniChainData.totalCdsDepositedAmount` is used.

#### What question he posed that lead him to this bug?
- he questioned with logical real example, if this calculation is correct or not. Here being mindfull is very important.

#### How to find it next time:
- Be more mindful when any calculations occurs and by default think it is incorrect, and then take different edge case examples in order to find if it gives the expected output or not
  

