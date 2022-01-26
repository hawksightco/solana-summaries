# Bonfida Token Vesting

[Github Repo](https://github.com/Bonfida/token-vesting)

## Instructions

1. `Init`
    1. Initializes an empty program account for the `token_vesting` program
    2. The vesting address is a program derived address (PDA) which is initialized
    3. By “empty”, what we mean is that the vesting address has not yet been assigned with the corresponding `mint_address` , `destination_token_address`, and `schedules`
    4. Accounts
        1. `system_program_account`
        2. `rent_sysvar_account`
        3. `payer`
        4. `vesting_accounht`
        5. `rent`
    5. Arguments (non-account)
        1. `seeds` : `[u8; 32]` - used to derive the vesting accounts address
        2. `number_of_schedules` : `u32` - number of release schedules for this contract to hold
    6. Account Validation
        1. `vesting_account` input is enforced to be a valid PDA key based on the `seeds` input
    7. Handling Logic
        1. Creates / initializes the vesting account ([link](https://github.com/Bonfida/token-vesting/blob/acb6901dbba3f6c0a564ab7d6e525b7c35c368d5/program/src/processor.rs#L28))
        2. Vesting account size is calculated
        3. Vesting account is created with the necessary rent balance transferred from the payer to the vesting account (PDA)
2. `Create`
    1. Creates a new vesting schedule contract, which maps a mint_address with a destination token address, and the schedules (which inform the time and amount of vesting)
    2. Accounts
        1. `spl_token_account`
        2. `vesting_account`
        3. `vesting_token_account`
        4. `source_token_account_owner` - Signer of the token transfer
        5. `source_token_account`
    3. Arguments (non-account)
        1. `seeds` : `[u8; 32]`
        2. `mint_address` : `Pubkey`
        3. `destination_token_address` : `Pubkey`
        4. `schedules` : `Vec<Schedule>`
    4. Account Validation
        1. Check that the vesting account is valid based on the seeds
        2. The source token account owner should be signing the transaction
        3. The vesting account should be owned by the token vesting program
        4. Simple vesting contract (SVC) has not yet been *created* with the input seed
        5. The vesting token account should be owned by the vesting account (PDA) itself
            1. The vesting token account can be the same across vesting contracts since there is no constraint that stops vesting token accounts from being used for multiple contracts
            2. However, there will always be enough funds in the vault token account since the total vesting amount per contract has to be fully deposited into the vesting token account before the contract is actually *created*
        6. The vesting token account should have no delegate authority (has not delegated its transfer authority to another address), and close authority
        7. The `source_token_account` has *enough* funds to send to the `vesting_token_account` based on the total vesting amount calculation
    5. Handling Logic
        1. Vesting account PDA data is written into with `VestingScheduleHeader` data: `destination_address`, `mint_address`, and `is_initialized`
        2. Total vesting amount is calculated. by getting the sum of the amounts for each of the schedules
            1. This is the total amount to be distributed from the vesting token account to the destination token account
            2. Note that each schedule contains information about the amount to be distributed, and the exact time that the distribution should be made
        3. Total vesting amount is transferred to the `vesting_token_account` from the `source_token_account`
            1. Note that the latter is the source of the tokens to be transferred to the vesting token account (which later in term will be transferring to the destination token account)
            2. Above means that a new simple vesting contract (SVC) can only be made when the total vesting amount is successfully transferred to the `vesting_token_account`
3. `Unlock`
    1. Unlocks a simple vesting schedule contract (SVC) - can only be invoked by the program itself
    2. Accounts
        1. `spl_token_account`
        2. `clock_sysvar_account`
        3. `vesting_account`
        4. `vesting_token_account`
        5. `destination_token_account`
    3. Arguments (non-account)
        1. `seeds` : `[u8; 32]`
    4. Account Validation
        1. Vesting account is a valid vesting account (PDA) based on the seeds
        2. SPL token program is the actual SPL token program
        3. Contract destination account (in the vesting account data field) matches the provided destination account
        4. The vesting token account is owned by the vesting account (PDA)
        5. Any of the vesting contract schedules have reached it release time (based on `release_time` ≥ `clock.unix_timestamp`
            1. `total_amount_to_transfer == 0` causes an error
    5. Handling Logic
        1. Unlock the schedules that have reached maturity
            1. I.e., for the schedules that have reached maturity (and not yet paid), the corresponding amounts are transferred from the vesting token account to the destination account
        2. Decode the vesting schedule data via `VestingScheduleHeader::unpack` (header struct) and `unpack_schedules` (array of schedules) from a byte array to readable rust objects
        3. For each schedule, check if the `release_time` is greater than or equal to the current `unix_timestamp` (derived from the `clock` account)
        4. If yes, then the corresponding amount is added to the `total_amount_to_transfer` (`total_amount_to_transfer += s.amount`)
            1. After adding the schedule amount to the amount set for transfer, the `amount` for that schedule is set to `0` (`s.amount = 0`)
            2. This schedule `amount` reset to `0` ensures that schedules cannot be paid out twice, so while the paid schedules are included in the loop, the amount added will remain to be `0` (with no effect to `total_amount_to_transfer`
            3. Such is also why the basis for “no vesting contract reaching the release time” is `total_amount_to_transfer` , because having a value of `0` necessitates having no unpaid mature schedule at that time
        5. `total_amount_to_transfer` is transferred from `vesting_token_account` to `destination_token_account`
        6. The schedule data is *updated* in order to reflect the. “zeroed” schedule amounts on the actual vesting account’s data on-chain
            1. "Reset released amounts to 0. This makes the simple unlock safe with complex scheduling contracts”
4. `ChangeDestination`
    1. Change the destination account of a given simple vesting contract (SVC) which can only be invoked by the present destination address of the contract.
    2. Accounts
        1. `vesting_account`
        2. `destination_token_account`
        3. `destination_token_account_owner`
        4. `new_destination_token_account`
    3. Arguments (non-account)
        1. `seeds` : `[u8; 32]`
    4. Account Validation
        1. Vesting account address is valid based on the seeds
        2. Contract destination account is similar to the input destination account
        3. Destination token account owner is signing the transaction
            1. This ensures that only the address receiving the vested tokens can authorize a change in the receiving address
        4. The provided owner should be the same as the owner of the current destination token account
    5. Handling Logic
        1. The current `destination_token_account` is replaced with a new `new_destination_token_account`