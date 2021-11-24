# Sunny

[Github Repo](https://github.com/arrowprotocol/arrow/tree/master/lib/sunny)

*This is a Sunny Anchor Client retrieved from [Arrow Protocol](https://github.com/arrowprotocol/arrow)*

## Instruction handlers

*These are the methods within the program module preceded by `#[program]`*

1. `init_vault`
2. `init_miner`
3. `deposit_vendor`
4. `stake_internal`
5. `unstake_internal`
6. `withdraw_vendor`
7. `withdraw_from_vault`
8. `claim_rewards`

## Contexts

*These are the structs that are preceded by `[#derive(Accounts)]` when using Anchor framework*

1. `InitVault`
    1. Accounts
        1. `pool`
        2. `owner`
        3. `vault`
        4. `payer`
        5. `system_program`
    2. Constraints
2. `InitMiner`
    1. Accounts
        1. `pool`
        2. `vault`
        3. `miner`
        4. `quarry`
        5. `rewarder`
        6. `token_mint`
        7. `miner_vault`
        8. `payer`
        9. `mine_program`
        10. `system_program`
        11. `token_program`
    2. Constraints
3. `WithdrawFromVault`
    1. Accounts
        1. `owner`
        2. `pool`
        3. `vault`
        4. `vault_token_account`
        5. `token_destination`
        6. `fee_destination`
        7. `token_program`
    2. Constraints
4. `ClaimRewards`
    1. Accounts
        1. `mint_wrapper`
        2. `mint_wrapper_program`
        3. `minter`
        4. `rewards_token_mint`
        5. `rewards_token_account`
        6. `claim_fee_token_account`
        7. `stake_token_account`
        8. `stake`
    2. Constraints
5. `QuarryStakeVendor`
    1. Accounts
        1. `vault_owner`
        2. `vault_vendor_token_account`
        3. `stake`
    2. Constraints
6. `QuarryStakeInternal`
    1. Accounts
        1. `vault_owner`
        2. `internal_mint`
        3. `internal_mint_token_account`
        4. `stake`
    2. Constraints
7. `QuarryStake`
    1. Accounts
        1. `pool`
        2. `vault`
        3. `rewarder`
        4. `quarry`
        5. `miner`
        6. `miner_vault`
        7. `token_program`
        8. `mine_program`
        9. `clock`
    2. Constraints