### Pubkey

```rust
/// Log a `Pubkey` from a program
pub fn log(&self) {
    #[cfg(target_os = "solana")]
    unsafe {
        crate::syscalls::sol_log_pubkey(self.as_ref() as *const _ as *const u8)
    };

    #[cfg(not(target_os = "solana"))]
    crate::program_stubs::sol_log(&self.to_string());
}
```

### Instructions

```rust
/// Returns a sibling instruction from the processed sibling instruction list.
///
/// The processed sibling instruction list is a reverse-ordered list of
/// successfully processed sibling instructions. For example, given the call flow:
///
/// A
/// B -> C -> D
/// B -> E
/// B -> F
///
/// Then B's processed sibling instruction list is: `[A]`
/// Then F's processed sibling instruction list is: `[E, C]`
pub fn get_processed_sibling_instruction(index: usize) -> Option<Instruction> {
    #[cfg(target_os = "solana")]
    {
        let mut meta = ProcessedSiblingInstruction::default();
        let mut program_id = Pubkey::default();

        if 1 == unsafe {
            crate::syscalls::sol_get_processed_sibling_instruction(
                index as u64,
                &mut meta,
                &mut program_id,
                &mut u8::default(),
                &mut AccountMeta::default(),
            )
        } {
            let mut data = Vec::new();
            let mut accounts = Vec::new();
            data.resize_with(meta.data_len as usize, u8::default);
            accounts.resize_with(meta.accounts_len as usize, AccountMeta::default);

            let _ = unsafe {
                crate::syscalls::sol_get_processed_sibling_instruction(
                    index as u64,
                    &mut meta,
                    &mut program_id,
                    data.as_mut_ptr(),
                    accounts.as_mut_ptr(),
                )
            };

            Some(Instruction::new_with_bytes(program_id, &data, accounts))
        } else {
            None
        }
    }

    #[cfg(not(target_os = "solana"))]
    crate::program_stubs::sol_get_processed_sibling_instruction(index)
}

// Stack height when processing transaction-level instructions
pub const TRANSACTION_LEVEL_STACK_HEIGHT: usize = 1;

/// Get the current stack height, transaction-level instructions are height
/// TRANSACTION_LEVEL_STACK_HEIGHT, fist invoked inner instruction is height
/// TRANSACTION_LEVEL_STACK_HEIGHT + 1, etc...
pub fn get_stack_height() -> usize {
    #[cfg(target_os = "solana")]
    unsafe {
        crate::syscalls::sol_get_stack_height() as usize
    }

    #[cfg(not(target_os = "solana"))]
    {
        crate::program_stubs::sol_get_stack_height() as usize
    }
}
```

### Program Stubs

```rust
fn sol_invoke_signed(
    &self,
    _instruction: &Instruction,
    _account_infos: &[AccountInfo],
    _signers_seeds: &[&[&[u8]]],
) -> ProgramResult {
    sol_log("SyscallStubs: sol_invoke_signed() not available");
    Ok(())
}
```

```rust
pub(crate) fn sol_invoke_signed(
    instruction: &Instruction,
    account_infos: &[AccountInfo],
    signers_seeds: &[&[&[u8]]],
) -> ProgramResult {
    SYSCALL_STUBS
        .read()
        .unwrap()
        .sol_invoke_signed(instruction, account_infos, signers_seeds)
}
```

```rust
/// Print the hexadecimal representation of the program's input parameters.
///
/// - `accounts` - A slice of [`AccountInfo`].
/// - `data` - The instruction data.
#[allow(dead_code)]
pub fn sol_log_params(accounts: &[AccountInfo], data: &[u8]) {
    for (i, account) in accounts.iter().enumerate() {
        msg!("AccountInfo");
        sol_log_64(0, 0, 0, 0, i as u64);
        msg!("- Is signer");
        sol_log_64(0, 0, 0, 0, account.is_signer as u64);
        msg!("- Key");
        account.key.log();
        msg!("- Lamports");
        sol_log_64(0, 0, 0, 0, account.lamports());
        msg!("- Account data length");
        sol_log_64(0, 0, 0, 0, account.data_len() as u64);
        msg!("- Owner");
        account.owner.log();
    }
    msg!("Instruction data");
    sol_log_slice(data);
}
```
