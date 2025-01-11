# Autonomint-Retrospective-WriteUp

## Approach 
- Core problem:
- What steps lead auditor to find this bug?
- What question he posed that lead him to this bug?
- How to find it next time:

## [H-1-696] - Users can withdraw liquidated collateral (through liq type 2) 

#### Core problem:

- This `depositDetail.liquidated` gets updated in liq type 1 but not in type 2, due to this in withdraw function this line `if (depositDetail.liquidated) revert IBorrowing.Borrow_AlreadyLiquidated();` will not revert for for liq type 2, and due to this user will be able to repay his debt and take out his collateral even after
he hot liquidated. Now when liq type2 happened at theat time collateral of borrower were already send to synthetics, but now when he is withdrawing the collateral,
here it means he is withdrawing someone elses collateral, its like stealing the funds of somone else, making loss of funds to other borrower, and even protocol.

- Also another problem is that a big amount of state changes did not occured in liq type2, that were occured in typ1 [here](https://github.com/sherlock-audit/2024-11-autonomint/blob/0d324e04d4c0ca306e1ae4d4c65f0cb9d681751b/Blockchain/Blockchian/contracts/Core_logic/borrowLiquidation.sol#L241-L277). This inconsistency will create a bad state for protocol. 

#### What steps lead auditor to find this bug?
- He simply compared the difference between liq type 1 and type 2, that's it.
 
#### What question he posed that lead him to this bug?
- When he saw that in liq type 2 `depositDetail.liquidated` state of borrower is not updated, he searched that exact state changes state in all contracts, and bound a bad state impact in function withdraw 

#### How to find it next time:
- Simply compare the two likely/opposite functions, and compare there state changes and if you found some inconsistency then try to see the bad state due to that inconsistency.


# [H-1-696] - Users can withdraw liquidated collateral (through type 2 liq) 
