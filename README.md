# Vault PDA - Solana Token Vault Program

A Solana vault program built with Anchor. This program implements a token vault system with share-based deposits and redemptions, along with protocol ownership management.

## Overview

The Vault PDA program demonstrates core Solana concepts including:
- **Program Derived Addresses (PDAs)** for deterministic account derivation
- **Token minting and burning** using SPL Token program
- **Share-based vault mechanics** with proportional deposit/redeem calculations
- **Protocol-level authority management** with ownership controls
- **Multi-vault architecture** supporting different underlying assets

## Program Structure

### Instructions

Located in `programs/vault-pda/src/instructions/`:

1. **`initialize`** - Initializes the protocol (one-time setup)
   - Creates the `ProtocolState` account to store the protocol owner
   - Creates the `VaultAuthority` PDA that serves as mint/burn authority for all vaults

2. **`initialize_vault`** - Creates a new vault for a specific underlying token
   - Creates a `Vault` account (PDA derived from underlying mint)
   - Creates a `share_mint` for vault shares
   - Sets the vault_authority as the mint authority

3. **`deposit`** - Deposit underlying tokens and receive vault shares
   - First deposit: 1:1 share minting
   - Subsequent deposits: Proportional shares based on vault state
   - Formula: `shares = (amount × total_shares) / total_assets`

4. **`redeem`** - Burn vault shares and withdraw underlying tokens
   - Proportional redemption based on share amount
   - Formula: `underlying = (shares × total_assets) / total_shares`

5. **`transfer_ownership`** - Transfer protocol ownership to a new owner
   - Updates the owner in ProtocolState
   - Validates ownership before transfer

### State Accounts

Located in `programs/vault-pda/src/state/`:

- **`ProtocolState`**
  - Stores the protocol owner
  - PDA seeds: `[b"protocol_state"]`
  - Size: 41 bytes

- **`VaultAuthority`**
  - Global authority for minting/burning vault shares
  - PDA seeds: `[b"vault_authority"]`
  - Size: 9 bytes

- **`Vault`**
  - Stores vault configuration for each underlying asset
  - Contains share_mint, underlying_mint, and vault_token_account references
  - PDA seeds: `[b"vault", underlying_mint]`
  - Size: 105 bytes

## Project Structure

```
solana-program/
├── programs/
│   └── vault-pda/
│       ├── src/
│       │   ├── lib.rs                      # Program entry point
│       │   ├── instructions/               # Instruction handlers
│       │   │   ├── initialize.rs
│       │   │   ├── initialize_vault.rs
│       │   │   ├── deposit.rs
│       │   │   ├── redeem.rs
│       │   │   ├── transfer_ownership.rs
│       │   │   └── mod.rs
│       │   ├── state/                      # State account definitions
│       │   │   ├── protocol_state.rs
│       │   │   ├── vault_authority.rs
│       │   │   ├── vault.rs
│       │   │   └── mod.rs
│       │   ├── constants.rs
│       │   └── error.rs
│       └── Cargo.toml
├── tests/
│   └── vault-pda.ts                        # Integration tests
├── Anchor.toml                             # Anchor configuration
└── package.json                            # Node dependencies
```

## Prerequisites

- [Rust](https://www.rust-lang.org/tools/install) (latest stable version)
- [Solana CLI](https://docs.solana.com/cli/install-solana-cli-tools) (v1.18.0 or higher)
- [Anchor](https://www.anchor-lang.com/docs/installation) (v0.31.1)
- [Node.js](https://nodejs.org/) (v18 or higher)
- [Yarn](https://yarnpkg.com/)

## Quick Start

1. Clone the repository:
```bash
git clone <repository-url>
cd solana-program
```

2. Install dependencies:
```bash
yarn install
```

3. Build the program:
```bash
anchor build
```

4. Run tests:
```bash
anchor test
```

## Running Tests

### Run all tests (with local validator):
```bash
anchor test
```

This command will:
- Start a local Solana test validator
- Build the program
- Deploy the program to the local validator
- Run the test suite
- Stop the validator

### Run tests with existing validator:
```bash
anchor test --skip-local-validator
```

Use this if you already have a validator running on `localhost:8899`.

## Test Suite

The test suite (`tests/vault-pda.ts`) includes:

1. Initializes the protocol
2. Creates an underlying token mint
3. Initializes a vault for the underlying token
4. Makes first deposit (1:1 share calculation)
5. Makes second deposit (proportional share calculation)
6. Redeems partial shares
7. Redeems all remaining shares

## Program ID

The program ID is declared in `lib.rs`:
```
8qsydpwMiRcFtJ8wrKkM4xrMMEWfnw2szibQGLgBw6KH
```

## Key Design Patterns

1. **PDA Derivation**: All program accounts use PDAs for deterministic addresses
2. **Token Interface**: Uses `anchor_spl::token_interface` for compatibility with Token and Token-2022
3. **Global Authority**: Single `vault_authority` PDA manages all vault share mints
4. **Vault Isolation**: Each underlying asset gets its own isolated vault
5. **Share Mechanics**: Proportional share calculations for fair deposits and redemptions

## License

ISC

## Resources

- [Anchor Documentation](https://www.anchor-lang.com/)
- [Solana Documentation](https://docs.solana.com/)
- [SPL Token Documentation](https://spl.solana.com/token)
