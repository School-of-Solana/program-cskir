# Project Description

**Deployed Frontend URL:** [TODO: Link to your deployed frontend]

**Solana Program ID:** 5rmQS8GUu872Xofm9LD1D8CUDvwdrdR32XLDbxcBcWED

## Project Overview

### Description
This project is a decentralized voting application ("Poll dApp") built on the Solana blockchain. It enables the creation and execution of simple yes/no polls.

### Key Features
- **On-chain poll creation**  
  Admin can create a poll with timestamps and a question (max 200 chars) and storage for results.
- **Voting Pass minting**  
  Admin mints a token that grants voting permission.
- **On-chain voter registration**  
  Each voter initializes a VoterState PDA tracking whether they have voted.
- **One vote per user enforced**  
  Voting burns the token and updates the poll state atomically. A voter may cast exactly one vote, consuming (burning) their Voting Pass. After the vote period ends, results (yes/no tallies) remain permanently on-chain.

  
### How to Use the dApp
#### 1. Connect Wallet
- Open the deployed dApp.
- Click **Connect Wallet** and connect Phantom (Devnet mode).

#### 2. Admin Workflow (with scripts)
1. Create Poll  
2. Enter question + timestamps  
3. Submit → Poll PDA is created  
4. Mint Voting Pass tokens to allowed voters  

#### 3. Voter Workflow
1. Initialize Voter State  
2. Request/receive Voting Pass  
3. When poll is active → click **Vote Yes** or **Vote No**  
4. Vote is recorded and token is burned  

#### 4. View Results
- After the poll ends, results appear and are permanently stored on-chain.

## Program Architecture
The dApp is built around three core components: 
- On-chain accounts:  defines two main account types, one representing the poll itself, and one representing an individual voter. 
- PDA-s
- SPL token based voting permission mechanism.

The Voting Pass token is an essential part of the design:
- Each eligible voter receives exactly 1 token from the admin
- The token must be burned during voting
This ensures:
- One-user–one-vote
- No double voting
- No token transfer  

The program implements five core instructions that define the full voting workflow. 

### PDA Usage
The program uses PDAs to guarantee deterministic, secure and non-colliding accounts.  

**PDAs Used:**
- **Poll PDA**: Derived from seeds `["pollpoll", poll_id]` - The PDA stores all metadata and results for each poll and prevent an admin from creating two polls with the same poll ID.
- **VoterState PDA**: Derived from seeds `["voter_state", poll_pubkey, voter_pubkey]` - Generate one PDA per poll, per wallet. The PDA tracks the voting status of a speific user, ensures each voter can vote exactly once.  

### Program Instructions

**Instructions Implemented:**
- **initialize (initialize_poll)**: Creates a new poll with a question and timestamps. Called by the admin.
- **start (start_poll)**: Marks a poll as active. Called by the admin.
- **mintPass (mint_voting_pass)**: Admin mints a single Voting Pass token to a voter.
- **addVoter (initialize_voter_state)**: Creates a VoterState PDA for the caller. Colled by any wallet.
- **vote (cast_vote)**: Called by any voter. Requirements:poll is active, poll is in time window, VoterState created, voter has not voted yet, voter owns exactly one Pass token) Execution: burn the token, update the poll's state, update the voterState.

### Account Structure
```rust
#[account]
pub struct Poll {
    pub admin: Pubkey,      // The admin who owns the poll 
    pub poll_id: u64,       // Unique poll id
    pub question: String,   // Poll's question
    pub start_ts: u64,      // Unix timestamp when poll's start 
    pub end_ts: u64,        // Unix timestamp when poll's end 
    pub is_active: bool,    // Poll's state
    pub voting_mint: Pubkey, // Voting pass mint 
    pub total_yes: u64,     // Total number of poll's yes votes
    pub total_no: u64,      // Total number of poll's no votes
}
#[account]
pub struct VoterState {
    pub poll: Pubkey,       // The current poll 
    pub voter: Pubkey,      // The voter who owns the state 
    pub has_voted: bool,    // the voter voted state
}
```

## Testing

### Test Coverage
Comprehensive test suite covering all instructions with both successful operations and error conditions to ensure program security and reliability.

**Happy Path Tests:**
- **Initialize Pool**: Successfully creates a new pool account with correct initial values
- **Create pass mint**: Successfully create pass mint
- **Start poll**: Successfully start poll
- **Initialize Voter State**: Successfully creates a new voter state with correct initial values
- **Vote**: Successfully vote with poll state updates



**Unhappy Path Tests:**
- **Initialize Poll Duplicate**: Fails when trying to initialize a poll that already exists
- **Initialize Poll Long Question**: Fails when trying to initialize a poll that questins above limit
- **Initialize Poll Invalid time window**: Fails when trying to initialize a poll that start is later then end
- **Unauthorized start poll**: Fails when not the admin trying to start a poll
- **Start poll Invalid time window**: Fails at start in case of out of time window
- **Start poll Invalid time window**: Fails at start in case of out of time window
- **Initialize Voter State Duplicate**: Fails when trying to initialize a voter state that already exists
- **Double Vote**: Fails double voting
- **Vote Inactive poll**: Fails vote for inactive poll
- **Vote Out of time window**: Fails vote for out of time window of the poll
- **Unauthorized Vote sign**: Fails when other try to sign a voter's vote  
- **Unauthorized Vote pass**: Fails when voter try to use other's voter's pass 
- **Unauthorized Vote state**: Fails when voter try to use other's voter's state 

### Running Tests
```bash
anchor test
```

### Additional Notes for Evaluators

This was my first Solana dApp and the learning curve was steep!