# Progress for various parts of Ignis

At the moment Ignis code is not ready to be open for a public (but will be) and is highly monolithic. Nevertheless, we can report some progress on various parts

- [ ] State keeper
  - [x] Process the current state
  - [x] Save the state
  - [x] Prepare information for prover and commitments 
  - [ ] Rollbacks and reorgs support   
- [ ] zkSNARKs
  - [ ] zkSNARK statements (what's proved)
    - [x] Register
    - [x] Transfer
    - [x] Exit
    - [ ] Emergency exit    
  - [x] Prover
    - [x] Proof generation
    - [x] Proofs are accepted by Ethereum network
  - [ ] Trusted setup
    - [x] Full in-memory powers of tau
    - [ ] Memory constrained powers of tau for public ceremony 
- [ ] Mempool
  - [x] Accept and organize transactions
  - [x] Lifetime monitoring
  - [x] Reorganize after new block assembly attempt
  - [ ] Optimize
  - [ ] Save mempool state to disk  
- [ ] Contract
  - [x] Registration
  - [x] Partial exits
  - [x] Complete exits
  - [ ] Emergency mode
  - [ ] Multioperator mode    
  - [x] Public key verification - basic (point is on curve)
  - [ ] Public key verification - advanced (point is in a correct group) - may be not required