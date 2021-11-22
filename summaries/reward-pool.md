# Reward Pool (Step Finance)

[Github Repo](https://github.com/hawksightco/reward-pool)

## Instruction handlers

*These are the methods within the program module preceded by `#[program]`*

1. `update_rewards`
    1. `fn` that updates the values for `reward_a_per_token_pending` (by adding the earned amount to the pending amount), and the `reward_a_per_token_complete`
    2. `reward_per_token`
        1. `fn` that returns the total rewards available for each token
    3. `earned`
        1. `fn` that returns the total amount of rewards earned
2. `reward_pool`
    1. `mod` that contains methods for the creation, and management of a "reward pool" for staking
    2. `initialize_pool`
        1. `fn` that initializes a reward pool while setting key accounts related to the ff
            1. Authority
            2. Staking token
            3. Rewards (up to 2 token mints)
            4. Reward duration & time
            5. Reward rates
            6. Rewards stored
            7. Number of stakers
    3. `create_user`
        1. `fn` that creates a user for this pool while setting the following
            1. Pool key
            2. Owner key
            3. Reward per token completed (set at 0)
            4. Reward per token pending (set at 0)
            5. Balance staked (set at 0)
            6. Updates the number of "staked users" for the pool (so the number of stakers for a pool are tracked)
    4. `pause`
        1. `fn` that pauses a staking pool
    5. `unpause`
        1. `fn` that unpauses a staking pool
    6. `stake`
        1. `fn` that stakes an amount to the staking pool for a specific user
        2. `update_rewards` with the updated amount staked (`total_staked`)
        3. `transfer` the tokens to be staked from the user to the staking vault
    7. `unstake`
        1. `fn` that unstakes an amount from the staking pool for a specified user
        2. `update_rewards`
            1. Update rewards right before unstaking so that earned rewards are reflected before the total balance staked is reduced
        3. Reduce the balance staked by the amount being unstaked (`spt_amount`)
        4. `transfer` the tokens to be unstaked from the pool vault to the unstaking user
    8. `authorize_funder`
        1. `fn` that authorizes a pubkey as a funder (one that funds the staking vault)
        2. An error message is returned if the pubkey passed is either already the pool authority,  or one of the already authorized funders
    9. `fund`
        1. `fn` that adds funds to either the Reward A vault, and/or the Reward B vault
        2. Before performing the transfer, `update_rewards` is called to ensure that rewards are properly accounted for before the rewards balances are increased
        3. Next a check is made on whether the reward period has already ended
            1. If yes, the reward rate is based solely on the current reward amount divided by the reward duration
            2. If no, the remaining duration is calculated, and used to derive the "leftover" rewards (by multiplying the remaining duration by the reward rate)
        4. Then, the reward rate is updated with the reward amounts that include the leftover rewards.
        5. Lastly, the reward tokens are transferred from the funder's reward token account to the staking reward vault account, and the end timestamp is updated.
    10. `claim`
        1. `fn` that claims the staking rewards that a specific user is entitled to
        2. First, the total staked amount is calculated, and the rewards are updated using the `total_staked` amount via `update_rewards`
        3. Then, it checks if there are any rewards pending (earned but unclaimed) based on the updated rewards
            1. If yes, then all the pending rewards are transferred to the user from the staking rewards vault
            2. If no, then no rewards are claimed
    11. `close_user`
        1. `fn` that removes a user from the staking pool
        2. Effectively, the `user_stake_count` is just reduced by 1
    12. `close_pool`
        1. `fn` that closes a staking pool, which involves closing the vaults (token accounts) that it owns
        2. The remaining `amount` in the `staking_vault` is transferred to the `staking_refundee`
        3. The `staking_vault` is closed using the `close_account` instruction of `spl_token`
        4. The remaining `amount` in the rewards token vaults are also transferred to the `reward_vault` for the corresponding reward token (a & b)
        5. Then, the rewards token vaults are closed, again via the `close_account` instruction of `spl_token`

## Contexts

*These are the structs that are preceded by `[#derive(Accounts)]` when using Anchor framework*

1. `InitializePool`
    1. Accounts
        1. `x_token_pool_vault`
        2. `x_token_depositor`
        3. `x_token_deposit_authority`
        4. `staking_mint`
        5. `staking_vault`
        6. `reward_a_mint`
        7. `reward_a_vault`
        8. `reward_b_min`
        9. `reward_b_vault`
        10. `pool_signer`
        11. `pool`
        12. `token_program`
    2. Constraints
        1. 
2. `CreateUser`
    1. Accounts
        1. `pool`
        2. `user`
        3. `owner`
        4. `system_program`
    2. Constraints
3. `Pause`
    1. Accounts
        1. `x_token_pool_vault`
        2. `x_token_receiver`
        3. `pool`
        4. `authority`
        5. `pool_signer`
        6. `token_program`
    2. Constraints
4. `Unpause`
    1. Accounts
        1. `x_token_pool_vault`
        2. `x_token_depositor`
        3. `x_token_deposit_authority`
        4. `pool`
        5. `authority`
        6. `pool_signer`
        7. `token_program`
    2. Constraints
5. `Stake`
    1. Accounts
        1. `pool`
        2. `staking_vault`
        3. `user`
        4. `owner`
        5. `stake_from_account`
        6. `pool_signer`
        7. `token_program`
    2. Constraints
6. `FunderChange`
    1. Accounts
        1. `pool`
        2. `authority`
    2. Constraints
7. `Fund`
    1. Accounts
        1. `pool`
        2. `staking_vault`
        3. `reward_a_vault`
        4. `reward_b_vault`
        5. `funder`
        6. `from_a`
        7. `from_b`
        8. `pool_signer`
        9. `token_program`
    2. Constraints
8. `ClaimReward`
    1. Accounts
        1. `pool`
        2. `staking_vault`
        3. `reward_a_vault`
        4. `reward_b_vault`
        5. `user`
        6. `owner`
        7. `reward_a_account`
        8. `reward_b_account`
        9. `pool_signer`
        10. `token_program`
    2. Constraints
9. `CloseUser`
    1. Accounts
        1. `pool`
        2. `user`
        3. `owner`
    2. Constraints
10. `ClosePool`
    1. Accounts
        1. `refundee`
        2. `staking_refundee`
        3. `staking_a_refundee`
        4. `reward_b_refundee`
        5. `pool`
        6. `authority`
        7. `staking_vault`
        8. `reward_a_vault`
        9. `reward_b_vault`
        10. `pool_signer`
        11. `token_program`
    2. Constraints