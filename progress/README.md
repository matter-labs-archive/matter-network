# Ignis Plasma Progress Tracker

At the moment Ignis code is not ready to be open for a public (but will be) and is highly monolithic. Nevertheless, we can report some progress on various parts. Please note that some points are "idealistic" in a sense that it may be not required for a final solution itself, but may provide a lot of additional benefits (like Universal Circuits)

# Ignis Progress Tracker

## SNARKs

### Circuits (what's proved)

- [x] Register
- [x] Transfer
- [x] Exit
- [ ] Emergency exit

### Prover

- [x] Proof generation
- [x] Proofs are accepted by Ethereum network

### Trusted setup

- [x] Full in-memory powers of tau
- [ ] Memory constrained powers of tau for public ceremony

### Production

- [ ] Base code cleanup and refactoring
- [ ] Universal circuits

### Extensions

- [ ] Atomic swaps
- [ ] Multisig

## Server

### Mempool

- [x] Accept and organize transactions
- [x] Lifetime monitoring
- [x] Reorganize after new block assembly attempt
- [ ] Optimizations
- [ ] Transaction persistence

### Networking

- [ ] P2P network protocol

### State keeper

- [x] Process the current state
- [x] Save the state
- [x] Prepare information for prover and commitments 
- [ ] Rollbacks and reorgs support

### Prover/committer pipelines

- [ ] Multi-instance management
- [ ] Transactions manager

## Smart contracts

### Base functionality

- [x] Registration
- [x] Partial exits
- [x] Complete exits
- [ ] Emergency mode

### Multi-Operator Model

- [ ] Security deposits
- [ ] Operator selection mechanism

### Production

- [ ] Partial block commitments
- [x] Public key verification - basic (point is on curve)
- [ ] Public key verification - advanced (point is in a correct group) - may be not required

## Client SDK

- [ ] Rust API
- [ ] WASM transcompilation
- [ ] JavaScript SDK
- [ ] Android SDK
- [ ] iPhone SDK
